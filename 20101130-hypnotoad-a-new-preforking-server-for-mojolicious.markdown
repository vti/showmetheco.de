Title: Hypnotoad: A new preforking Perl server for Mojolicious
Tags: Perl, Mojolicious, server, preforking
Comments: no

From the documentation:

[Mojo::Server::Hypnotoad](https://metacpan.org/pod/Mojo::Server::Hypnotoad) is a full featured UNIX optimized preforking async
io HTTP 1.1 and WebSocket server built around the very well tested and
reliable [Mojo::Server::Daemon](https://metacpan.org/pod/Mojo::Server::Daemon) with `TLS`, `Bonjour`, `epoll`, `kqueue`
and hot deployment support that just works.

[cut]

This is a new preforking server that we all have been waiting for after the old
preforking server was deprecated.

This server has some really unique features:

- WebSocket support

    While writing a standalone WebSocket server isn't that difficult
    ([http://showmetheco.de/articles/2010/11/timtow-to-build-a-websocket-server-in-perl.html](http://showmetheco.de/articles/2010/11/timtow-to-build-a-websocket-server-in-perl.html)),
    preforking should be done with care and quality that [Mojolicious](https://metacpan.org/pod/Mojolicious) always brings to you.

- Hot deployment

    With the help of UNIX signals, you can control number of workers, upgrade
    without losing any incoming connections, gracefully shutdown the server and
    workers.

- Advanced configuration

    From a configuration file (normal Perl hash) you can control queue size for
    listen, workers number, maximum number or parallel clients, user and group,
    reverse proxy support, various timeouts, lock and pid files and many more.

Check it out on GitHub [http://github.com/kraih/mojo](http://github.com/kraih/mojo)!
