Title: Why I chose to build a Plack framework instead of using Mojolicious
Tags: Perl, Mojolicious, Plack, framework
Comments: no

I've invested a lot of time in [Mojolicious](https://metacpan.org/pod/Mojolicious) development (a so called "next
generation web framework for the Perl programming language"). Of course I didn't
commit any revolutionary patches, or affected it's architecture somehow, but I
tried to use it in every scenario I could think of, reported bugs, proposed a
few feature requests. So why I decided to switch to [Plack](https://metacpan.org/pod/Plack) and build my own
web framework?

[cut]

## Introduction

I will try to be objective and try not to mention any personal or community
issues. The main reason of this article is not to convince anyone but to explain
how I became a `pragmatic` programmer.

Before I start here are brief descriptions of [Mojolicious](https://metacpan.org/pod/Mojolicious) and [Plack](https://metacpan.org/pod/Plack)
respectively.

[Mojolicious](https://metacpan.org/pod/Mojolicious) is a framework that claims to be "the next generation web
framework for the Perl programming language". Among various buzz words in its
features it has RESTful routes, plugins, Perl-ish templates, sessions, signed
cookies, HTTP/1.1 server/client implementation, WebSocket support with async IO,
JSON and XML/HTML5 parser with CSS3 selectors.

[Plack](https://metacpan.org/pod/Plack) is Perl Superglue for Web frameworks and Web Servers. It contains
middleware components, a reference server and utilities for Web application
frameworks.

It would be silly to compare [Mojolicious](https://metacpan.org/pod/Mojolicious) and [Plack](https://metacpan.org/pod/Plack), but I am comparing
[Plack](https://metacpan.org/pod/Plack) to something what [Mojo](https://metacpan.org/pod/Mojo) used to be: a set of tools for building your
own framework.

## Wrong assumption

One of the claimed [Mojolicious](https://metacpan.org/pod/Mojolicious) strengths is its 'Web in a box!' motto. But
for me it is the biggest weakness. It is a highly coupled, monolithic,
one-way-to-do-it, inflexible (because it's not supposed to be) framework.

    More code - more bugs.

Having everything just right there looks like a good thing, but it
works like a vendor lock-in. Once you realize that you want to change how
dispatching works, how sessions are handled, how rendering or a low level
http parsing is done - you are screwed, because you are 'holding it wrong'.

Once you realize that you don't need Async IO and want to minimize a memory
footprint or startup time, switch off everything that you don't need for a
simple CGI application - you're screwed. And don't forget:

    More code - more bugs.

## Implementation

Object-oriented programming is like 30 years old. Sorry, [Mojolicious](https://metacpan.org/pod/Mojolicious) is not
there. If you see a nice looking candy wrapper it doesn't mean there is a candy
inside. 150+ lines long methods, deep not obvious inheritance, absence of
exception handling, global namespace for saving everything from system settings
to user variables, highly coupled components.

Ever wondered why you need a WebSocket support if you don't need it, and why
there are 27 checks like:

    $tx->is_websocket

in subroutines that deal with routes and controllers?

Have you read Perl Best Practices or Clean Code? Do you like self explanatory
code and hate overused comments? Do you think that tests must be as clean as the
main code? Do you think that tests don't prove anything, except that they test
what you actually test? Forget about it.

From [Mojo::Util](https://metacpan.org/pod/Mojo::Util):

    # Increment
    $i++;

## Benefits to Perl community

[Mojolicious](https://metacpan.org/pod/Mojolicious) has a lot of parts. But can you use them if you aren't using
[Mojolicious](https://metacpan.org/pod/Mojolicious)? No. Everything or nothing.

How about releasing routing, templates, DOM and JSON parsing etc as a separate
modules so everybody could benefit? No. Everything or nothing.

## Plack alternative

[Plack](https://metacpan.org/pod/Plack) is a good example for collection of modules (bricks) that can be easily
combined together.

Middleware processing (known also as a filter chain) is the most flexible way for
response <-> request handling. Separated classes that wrap your application can add
new functionality, control the flow, perform pre- and postprocessing actions.

Handlers or server adapters add a transparent support for various web servers,
eliminating the need to implement, test and maintain them.

## Writing a Plack web framework

So what do you need in order to write a [Plack](https://metacpan.org/pod/Plack) web framework?

HTTP parsing? Sessions? Logging? Debug panel? Static files? Exception handling?
Protection from CSRF? Gzip? JavaScript or CSS preprocessing? It's already there.
In separate and pluggable middlewares.

Need route dispatching? Rendering? Configuration loading? Implement it as a
middleware and release so everybody can benefit.

Don't like [Moose](https://metacpan.org/pod/Moose)? Like [Moose](https://metacpan.org/pod/Moose)? Don't like Async IO? Like Async IO? Write
your framework and use what you like.

## Conclusion

There is no tool that does everything and does it well. But there can be a lot
of small tools that do small things and do them well. By combining small tools
a perfect fit for solving the current problem can be found.
