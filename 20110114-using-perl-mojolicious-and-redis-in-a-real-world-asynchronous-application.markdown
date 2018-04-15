Title: Using Perl, Mojolicious and Redis in a real world asynchronous application
Tags: Perl, Mojolicious, Redis, async
Author: und3f
Comments: no

[Mojolicious](https://metacpan.org/pod/Mojolicious) is a growing modern Perl web framework which is ready for
building real world applications. Today we will consider architecture of
[http://check-host.net](http://check-host.net) which is built on top of [Mojolicious](https://metacpan.org/pod/Mojolicious) modules.

Redis is a powerful key-value database that allows you to create fast and
lightweight applications. You can learn more about using Redis in Perl from
other my article
[http://showmetheco.de/articles/2010/11/mojox-redis-an-asynchronous-redis-implementation-for-mojolicious.html](http://showmetheco.de/articles/2010/11/mojox-redis-an-asynchronous-redis-implementation-for-mojolicious.html).
Building asynchronous applications allows you to avoid using threads, so such
applications tends to use less memory and human power to deal with all their
complexity.

[cut]

## Introduction

[http://check-host.net](http://check-host.net) is a collection of online tools for checking
accessibility of different aspects of the host: ping, TCP port connection, HTTP
request and DNS records check. Checks are performed from the different
geographical locations.

## Architecture

Check-host consists of 3 different parts: a web site (front-end), a work
scheduler (back-end which schedules check requests to workers) and multiple
workers (running on many servers around the world).

Multiple workers represent the GRID system
([http://en.wikipedia.org/wiki/Grid\_computing](http://en.wikipedia.org/wiki/Grid_computing)).

All parts are written in Perl.

Web site is written in a modern Perl web framework - [Mojolicious](https://metacpan.org/pod/Mojolicious). Work
scheduler receives user's check requests from the web site and sends tasks to
the workers (considering check request queue). Multiple workers are placed on
servers in different geographic locations. All parts of check-host are
asynchronous and use [Mojo::IOLoop](https://metacpan.org/pod/Mojo::IOLoop) as an event loop. Key-value database Redis
is used to store the data checks and communication of front-end and back-end.
To work with Redis [MojoX::Redis](https://metacpan.org/pod/MojoX::Redis) module is used.

After a user requests a check it is stored with a key being `check:requests:ID`,
where ID is an auto incremented identificator. Also the check request is added
to the checks queue. Checks queue is a key of `List` type that can work as a
pipe with commands `blpop` and `rpush` commands.

Communication between workers and work schedulers is based on the simple serial
protocol with encoding data in `JSON` format. All the communication is secured with
a `TLS` encryption.

An example of work scheduler <-> worker communication:

Work scheduler -> worker

    ["ping", "google.com", 4]

The worker pings google.com four times and replies

    [["OK", 0.01], ["TIMEOUT"], ["OK", 0.02], ["OK", 0.03]]

The work scheduler stores the worker's reply and sends the next command.

Workers perform multiple checks, some of them use the standard [Mojolicious](https://metacpan.org/pod/Mojolicious) parts
(like [Mojo::Client](https://metacpan.org/pod/Mojo::Client) or sockets of [Mojo::IOLoop](https://metacpan.org/pod/Mojo::IOLoop)), for others additional
modules were written (for example [MojoX::Ping](https://metacpan.org/pod/MojoX::Ping)).

## Advices on developing serious applications with Mojolicious

Always write tests for the most parts of your application. This way you won't
be afraid of updating [Mojolicious](https://metacpan.org/pod/Mojolicious) ([Mojolicious](https://metacpan.org/pod/Mojolicious)' API changes often). Also
that will help you to make refactoring easy when you will be ready for that.

If you write a big closed application - try to release some of its modules as
open source. That will help you to find some bugs in it :)

Never use EXPERIMENTAL API of Mojolicious. That API could be changed two or
three times before stabilization.

## More information

More information can be found on the website [http://check-host.net/about](http://check-host.net/about).

## About the author

Author of this post is und3f. You can follow him on Twitter
[http://twitter.com/und3f](http://twitter.com/und3f) and/or GitHub [http://github.com/und3f](http://github.com/und3f).
