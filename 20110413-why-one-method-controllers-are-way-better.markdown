Title: Why one-method controllers are way better
Tags: Perl, MVC, controller

I see all these Perl frameworks that have controller classes with actions. And
sometimes I see those that have tons of configuration stuff and implementation
details, that could be just thrown away by using plain Perl.

[cut]

This is the common design: dispatch request, create a controller class instance,
find an appropriate method, call it.

So here you have a bunch of potential problems, that you have to face in order
to `hold it right`:

You can't map special Perl methods like `new`, `DESTROY` etc.

You can't map methods that are provided by a framework's base controller class
or you have to create your actions with a special prefix, attribute etc (yak!).

Forget about code reuse, since all your controllers are going to be different,
hard to inherit and decorate, impossible to combine.

You can create some kind of a convention when you have a base controller with
default CRUD methods like `add`, `remove` etc. But what if you don't need
`add` method? Should you overwrite it as an empty sub? You can use roles
(mix-ins) of course and add only needed methods in every controller, but then..
why do you put all the actions into one class?

The solution is simple. Use only one method in a controller. Let it be always a
method `run`. Then you can map whatever you want to just a single class.

    $routes->add_route('/articles/new' => 'MyController');

    package MyController;

    use base 'MyFramework::Controller';

    sub run {
    }

    1;

The advantages our obvious:

- Easy to maintain
- Code is cleaner
- Easy to inherit, decorate and combine
- Easy to test just one action
- No need to worry about method names
- Incapsulation

Some of the disadvantages:

- More classes (but they are simple)
- Maybe an overkill for simple apps

It definitely worths a try.
