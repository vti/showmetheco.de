Title: PocketIO - realtime applications for Plack
Tags: Perl, socketio

[PocketIO](https://metacpan.org/pod/PocketIO) is a [SocketIO](http://socket.io) port from Node.JS to Perl. It
allows you to write realtime web applications without worring about specific
browsers features: from long-polling to WebSockets.

[PocketIO](https://metacpan.org/pod/PocketIO) is built on top of [AnyEvent](https://metacpan.org/pod/AnyEvent) and [Plack](https://metacpan.org/pod/Plack), runs smoothly on
[Twiggy](https://metacpan.org/pod/Twiggy). This way it can be easily combined with other nonblocking [Plack](https://metacpan.org/pod/Plack)
apps.

[PocketIO](https://metacpan.org/pod/PocketIO) can be scaled using [Redis](http://redis.io) pub/sub infrastructure.

Proved to work well in production with 100-150 simultaneous connections. Works
just fine with TLS/SSL (via [App::TLSme](https://metacpan.org/pod/App::TLSme)) too.

Check it out. On [Github](http://github.com/vti/pocketio) or now on CPAN.
