Title: Routes::Tiny tips and tricks
Tags: Perl

[Routest::Tiny](http://metacpan.org) have improved a lot since the first release in 2011. Here I collected some tips and
tricks that maybe are not that obvious from the documentation.

Below are the practical examples from the working [Kritika](https://kritika.io) code.

[cut]

# Named routes

This is the best thing about Routes::Tiny. You can not only parse the paths but also build them! I always use named
routes. For example:

```
$routes->add_route( '/', name => 'index', method => 'GET' );
```

Now anywhere in the controller, template, helper I can just use `build_path('index')` and will get the correct path (or
absolute url if I want). This way changing the way your URLs look like is a matter of changing them only in one place
where they were declared.

# Via routes

Imagine a very common scenario. You have a base path `/users/:user_id` and now you want to plug in an action to this URL
like `/users/:user_id/settings`. And inside of the controller attached to `/users/:user_id` you're doing some logic that
has to be replicated in the new settings, something like loading a user and checking permissions. Now what you want to
do is to create a `via` or `proxy` action that should will be always called when accessing a nested path.

The solution to this is to use `arguments` inside of the nested routes. For example:

```
my $profile = $routes->add_route('/profiles/:profile_id',
    'arguments' => {via => 'user_profile_via'});

$profile->add_route('/',       name => 'user_profile');
$profile->add_route('/save',   name => 'user_profile_save');
$profile->add_route('/delete', name => 'user_profile_delete');
```

Now when doing a routes match we can get `via` argument and call the correct action.

But what if you want to have many `via` actions? That's where you can use `+arguments`. Which is the same as `arguments`
but instead of replacing the previous value it is pushed creating an array.

Now you just to have to do something like this:

```
foreach my $via (@$vias) {
    my $via_action = $self->_build_action($via, $env);

    die 'not found' unless $via_action;

    $via_action->run;

    ...
}
```

Are you using Routes::Tiny? Do you have your own tips?
