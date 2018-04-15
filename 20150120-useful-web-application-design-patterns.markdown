Title: Useful web application design patterns
Tags: Perl

I have collected the most useful web application design pattern that I use in
web applications I code at work and also for my personal projects. They are
presented in a way of a problem and a solution.

[cut]

All of the examples are for PSGI/Plack based web applications. But I am sure
they can be used in web frameworks as well.

# Filter/Middleware oriented design

Many of the web application functionality can be separated into so called filter
chains or middleware in terms of Plack. But it can be not just some light
request/response altering, but rather a complete subsystem. Since middleware
are already decoupled it's easy to build simple and effective solutions.

Let's take a look at the most common web application task: request dispatching.
We want to take an HTTP-request and map it onto the Perl class, call it and then
get a template rendered.

The naive solution might be to do that in a single place like most of the web
frameworks do.

```
sub {
    my $env = shift;

    my $path_info = $env->{PATH_INFO};

    my $controller_class = $path_info

    return [404, [], ['Not found']] unless try_load_class $controller_class;

    my $res = $controller_class->new->run($env);

    return [200, [], $res];
}
```

It's all good and nice until you want to use smth like routes here. Then you
would want to add authorization, user roles, default templates etc.

You will have to move everything to controller thus making them fat and
very hard to test, since you will have to mock the whole environment.

The better and simpler approach is to separate responsibilies of the request
phrases into middleware. For example:

```
enable 'RequestDispatcher';
enable 'ActionDispatcher';
enable 'ViewDisplayer';
```

Where `RequestDispatcher` takes the HTTP-request and transforms it into
understandable for web application object. For example you can use routes there.
Then if the correct route is found `ActionDispatcher` creates controller and
runs it. `ViewDisplayer` renders the appropriate template.

As a side effect you have a possibility to rendere a template without calling
a controller, which is needed in many web applications.

Later on you decide you need an authorization. Just write another middleware:

```
enable 'RequestDispatcher';
enable 'User';
enable 'ACL';
enable 'ActionDispatcher';
enable 'ViewDisplayer';
```

Where `User` middleware loads a user from a cookie for example and `ACL` checks
if a current user has a role for executing this controller.

Controllers are kept thin (there is no need to check user rights everywhere),
every middleware follows Single Responsibility Principle and everything is
configurable and extendible without effecting other parts of the system.

I die a little inside when I see this:

```
package MyController;
use base 'WebFramework::Controller';

sub action {
    my $self = shift;

    my $user = User->load($self->req->cookie('user_id'));
    if ($user->role eq 'admin') {
        ...
    }
    else {
        return $self->403_error;
    }
}
```

When using middleware you move all of this stuff into one place and clean up
your controllers and tests for them.

# Single method controllers

I have already
[written](http://showmetheco.de/articles/2011/4/why-one-method-controllers-are-way-better.html)
about this. But I have to mention it here too.

Many web frameworks create controllers that have several methods. For example,
we have a controller called `Post` and then it has internal CRUD methods like
`create`, `update` etc. While this may seem like a good idea but a better idea
is to keep it simple and create a separate package for every action. This will
also help to follow Single Responsibility Principle and make testing a lot
easier.

Also this way you can use composition separately. It is easier to apply
a Role/Mixin or inherit from a base controller if you have just one method to
worry about.

# Request scopes

In web applications most of the time you have objects with two life cycles. The
first one has to be available all the time and you want to build it once. The
second one has to be there just for a request.

When using Plack you can create application objects or as I call them services
and env objects that are created during a request.

For example, we have a mail object that can send mail. We want to be available
in controllers and don't want to create and configure it every time. Thus the
controller will have just two attributes: services and env.

```
package MyAppController;

sub new {
    my $class = shift;
    my (%params) = @_;

    my $self = {};
    bless $self, $class;

    $self->{env}      = $params{env};
    $self->{services} = $params{services};
}

sub service { shift->{services}->service(@_) }
sub req     { Plack::Request->new(shift->{env} }
```

Controller doesn't have to be any more complex than that.

`env` is also used for passing objects between middleware, thus you can put
here template variables and so on.

# Service containers

In the previous chapter I mentioned services. How does one configure them for
the whole application? You can write a container, a simple hash or anything
holding the services that you pass to other components of the system. Sometimes
a simple IoC container is a good choice. This way you have smth like:

```
package MyApp;
use base 'WebFramework';

sub startup {
    my $self = shift;

    my $services = $self->services;

    $services->register('mail' => MyMailObject->new);

    ...
}

sub to_psgi_app {
...
}
```

# Pull templates

Pull templates or active templates are templates that can call model methods
from the templates. This could sound like a crazy idea, but if you do it right
it has enourmous benefits.

Say for example you have to know in every template if user is logged in or not.
Do you really put `user` variable in every controller for the template to be
happy? Or would you rather call from the template special method to know if the
user is logged in?

These special methods are usually called helpers. And if you keep implementing
helpers in such a way that they just have a read only access to whatever they
have access to, you are going to be allright.

For example an active template can look like:

```
% if ($helpers->acl->is_user)
<span>Logged in</span>
% } else {
<span>Anonymous</span>
% }
```

And a helper can look like:

```
package Helpers::ACL;
use base 'WebFramework::Helper::Base';

sub is_user {
    my $self = shift;

    return $self->env->{'user'} ? 1 : 0;
}
```

Also when you don't have a controller and just want to render a template,
helpers are already there and registered in a separate middleware.

# Implementation

If you liked these patterns, you might want to take a look at
<http://github.com/vti/tu> and a real life web application example
<http://github.com/vti/threads>.
