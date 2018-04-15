Title: Stash, Registry and DI container
Tags: Perl, design patterns

It turns out that all web frameworks require some kind of a sharing mechanism
for passing variables to templates, holding configuration, database connection
etc. This common data should be accessed from everywhere: controllers, models,
templates, api endpoints, utility scripts and so on. Designing this "everything
in one" class can dramatically affect web framework flexibility, extensibility,
readability, maintainability and global warming.

[cut]

## Stash

The first naive approach is to put everything into one hash and call it `stash`.
Let's put there template names and variables, http statuses, errors,
configuration, helpers and so on. You end up with a mess. If you don't divide
stash keys by some kind of a namespace ('myapp.displayer.template'), which is
ugly, you end up creating fun hours of debugging for a developer who chose to
name his variable 'status' that "unexpectedly" conflicts with a system key.

Sounds not very promising. But there are many design patterns around. Let's try to
find one that satisfies our requirements: holds various kinds of objects that
are accessible from everywhere. One of these (so-called creational) patterns is
`Registry`.

## Registry

Basically `Registry` is a singleton (globally accessible) that holds objects of
any kind. Consider the following code:

    # somewhere when app is just created
    MyApp::Registry->set_config({foo => 'bar'});

    # later on
    my $config = MyApp::Registry->get_config;

Now we can access config everywhere. No need to pass objects around, no need
for manual config loading and parsing. It's all ready. Unfortunately as
everything global it has a downside. What if you wanted to have two registries
(running two instances of the same application)?

## Dependency Injection Container

Let's try another pattern called `Dependency Injection Container`, which is a
local `Registry` with `Dependency Injection` capabilities. `Dependency Injection`
is a pattern when you pass already created objects as dependencies to other
objects. So instead of:

    package Car;
    use Engine;

    sub start {
        my $engine = Engine->new;
        return $engine->start;
    }

    my $car = Car->new;
    $car->start;

You do:

    package Car;
    use Engine;

    sub start {
        return $self->{engine}->start;
    }

    my $engine = Engine->new;
    my $car = Car->new(engine => $engine);
    $car->start;

This may look like more work, but this way you get rid of a static `Engine`
dependency and can pass a modified object with the same interface without
modifying `Car` class.

    my $engine = AnotherEngine->new;
    my $car = Car->new(engine => $engine);
    $car->start;

This not only simplifies class runtime modification (you don't have to create
additional classes, just pass needed object), but also simplifies testing (using
mocks).

When you have many objects inside of a web framework, passing all of them around
is a lot of work indeed. But we can pass just one variable that holds all of our
dependencies and get any of them when we actually need.

So the `Dependency Injection Container` is a container that is populated with
objects that are used as dependencies when building other objects. We create a
new container, populate (or register) it with the dependencies we want to have
access to and then access them on demand.

Below is an example of a `Dependency Injection Container`.

Let's call an object inside of a container a `service`. We can pass already
created services but we can save some typing by passing just class names and
lazy initiating them on the first call:

    my $container = MyApp::DIContainer->new;
    $container->register(my_service => MyService->new);

    # or

    my $container = MyApp::DIContainer->new;
    $container->register(my_service => 'MyService');

To get a service from a container let's use `get_service` method. The objects
inside of a container are singletons (scoped to the container object), and are
created just once (that's by default, but you can have something like
`create_service` when you need a fresh service every time):

    # object MyService is created for the first time
    my $service = $container->get_service('my_service');

    # same MyService objects is returned
    $service = $container->get_service('my_service');

    # new MyService objects is returned every time
    $service = $container->create_service('my_service');

### Constructor Injection

Most of the time services require other objects or even other services (like
engine of the car requires a gas tank). During service creation the dependencies
can be passed as normal arguments to the constructor (`Constructor Injection`),
this can be automated:

    my $container = MyApp::DIContainer->new;
    $container->register(my_service => 'MyService');
    $container->register(
        other_service => 'OtherService',
        dependencies  => ['my_service']
    );

    my $other_service = $container->get_service('other_service');

Which is the same as:

    my $service = MyService->new;
    my $other_service = OtherService->new(service => $service);

### Setter Injection

Sometimes you don't want to pass dependencies to constructor, but instead call a
setter (`Setter Injection`). This can be implemented like this:

    my $container = MyApp::DIContainer->new;
    $container->register('other_service' => 'OtherService');
    $container->register(
        my_service   => 'MyService',
        dependencies => ['other_service'],
        setters      => {other_service => 'set_other_service'}
    );

After MyService creation method `set_other_service` will be called with
`other_service` as a parameter:

    my $other_service = OtherService->new;
    my $service = Service->new;
    $service->set_other_service($other_service);

Good job! You're still reading :)

DI Container can be a framework by itself. If you search CPAN you can find
[Bread::Board](https://metacpan.org/pod/Bread::Board) (see also [IOC](https://metacpan.org/pod/IOC), it's not maintained anymore but you can find
interesting links for further reading), [Peco::Container](https://metacpan.org/pod/Peco::Container) and others with many
different features like constructor overwriting, aliases etc. Once created and
configured container can be passed as a dependency injection to all objects
that need it.

### Usage example

Here is a real life example that can be found in a web framework:

    # somewhere in startup {} method
    my $container = DIContainer->new;
    $container->register('root' => '/etc/my_app.yml');
    $container->register(
        'config_loader' => 'YamlLoader',
        dependencies    => 'root'
    );
    $container->register(
        'config'     => 'Config',
        dependencies => 'config_loader'
    );
    $container->register('dbh' => 'Database', dependencies => 'config');

    # and so on...

    $app->set_container($container);

    # later somewhere in controller
    sub add_article {
        my $self = shift;

        my $container = $self->container;

        # all the required objects are created and passed
        # to the $dbh object automatically
        my $dbh = $container->get_service('dbh');
        $dbh->do('...');
    }

Clean, decoupled and convenient.
