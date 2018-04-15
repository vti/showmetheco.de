Title: May updates
Tags: Perl, CPAN
Comments: no

In case you use some of the following modules you might be interested in reading
what happened to them: App::TLSMe, Fliggy, Protocol::WebSocket,
Plack::Middleware::SocketIO, MojoX::Validator, Protocol::XMLRPC.

[cut]

- [App::TLSme](https://metacpan.org/pod/App::TLSme) is now on CPAN.

    [App::TLSMe](https://metacpan.org/pod/App::TLSMe) is an [AnyEvent](https://metacpan.org/pod/AnyEvent) TLS/SSL tunnel that transparently proxies
    encrypted requests to the non-SSL applications via INET or UNIX sockets.

- [Fliggy](https://metacpan.org/pod/Fliggy) is now on CPAN too.

    [Fliggy](https://metacpan.org/pod/Fliggy) is [Twiggy](https://metacpan.org/pod/Twiggy) with inlined Flash Policy server that eliminates the need
    of running a separate service on 843 port as root.

- [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) has an API change.

    Due to internal UTF-8 encoding/decoding sometimes when you don't need Perl
    strings and want to get just raw bytes you'd have to encode/decode manually and
    introduce unneedable overhead. Now you can get Perl string and raw UTF-8 decoded
    data separately by using specific methods. In [Protocol::WebSocket::Frame](https://metacpan.org/pod/Protocol::WebSocket::Frame)
    `to_string` now returns Perl strings instead of bytes and `to_bytes` does the
    opposite. `next` left without a change and returns Perl strings as is did, but
    a new method `next_bytes` returns raw UTF-8 data.

- [Plack::Middleware::SocketIO](https://metacpan.org/pod/Plack::Middleware::SocketIO) is now considered deprecated in favor of
[http://github.com/vti/pocketio](http://github.com/vti/pocketio).

    Middlewares are not suitable for the end-point applications. Because of this
    PocketIO is now built like [Plack::App::File](https://metacpan.org/pod/Plack::App::File). From now on you can just
    mount SocketIO application as any other Plack application.

        mount '/socket.io' => PocketIO->new(handler => sub { ... });

    PocketIO is already ahead of [Plack::Middleware::SocketIO](https://metacpan.org/pod/Plack::Middleware::SocketIO). In particular,
    various UTF-8 related bugs are fixed and connections are correctly closed.

- [MojoX::Validator](https://metacpan.org/pod/MojoX::Validator) is now [Input::Validator](https://metacpan.org/pod/Input::Validator).

    Since I wanted to use [MojoX::Validator](https://metacpan.org/pod/MojoX::Validator) in [Plack](https://metacpan.org/pod/Plack) applications I've got rid
    of [Mojolicious](https://metacpan.org/pod/Mojolicious) dependency, but not from [MojoX::Validator](https://metacpan.org/pod/MojoX::Validator) functionality.
    The new [Input::Validator](https://metacpan.org/pod/Input::Validator) is just the same thing with the same interface but
    CPAN friendly.

- [Protocol::XMLRPC](https://metacpan.org/pod/Protocol::XMLRPC) is refactored.

    [Protocol::XMLRPC](https://metacpan.org/pod/Protocol::XMLRPC) is a [XML::LibXML](https://metacpan.org/pod/XML::LibXML) based XMLRPC message parser/builder. Now
    it has an improved automatic type guessing similar to [JSON](https://metacpan.org/pod/JSON).

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 11:

    &#x3d;over without closing =back
