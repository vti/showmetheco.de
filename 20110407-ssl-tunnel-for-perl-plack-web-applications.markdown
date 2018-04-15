Title: SSL tunnel for Perl/Plack web applications
Tags: perl, ssl, tls, tunnel

Adding TLS/SSL support to your Perl web application could cause a headache if
it's not embedded in your web server. The most popular solution is to use an ssl
tunnel in front of your server that transparantly encrypts/decrypts messages. In
order to tell [Plack](https://metacpan.org/pod/Plack) application that TLS/SSL tunnel is used at least two
special HTTP headers `X-Forwarded-For` and `X-Forwarded-Proto` must be set.
The problem is that a well-known tunneling application `stunnel` does not
support `X-Forwarded-Proto` header...

[cut]

This is not a problem if you don't need to know if you run under TLS/SSL.  You
even don't have to care in your templates about `http://` and `https://`
because you can use these urls:

    <a href="//example.com/home">Home</a>

`//` will be automatically replaced by the current protocol.

The problem arrises when you have to use WebSockets for example (either raw or
as a part of Socket.IO suite). This way you have to change not only protocol url
to `wss://` but also WebSocket-specific HTTP headers.

In order to solve this problem I've written a simple TLS/SSL tunnel in
[AnyEvent](https://metacpan.org/pod/AnyEvent) that supports `X-Forwarded-Proto` headers and plays nicely with
[Plack::Middleware::SocketIO](https://metacpan.org/pod/Plack::Middleware::SocketIO). The source code is available on GitHub
[https://github.com/vti/app-tlsme](https://github.com/vti/app-tlsme). One thing that stops me from releasing it
on CPAN is a small bug in the current [AnyEvent](https://metacpan.org/pod/AnyEvent), but the patch that fixes the
problem will be shipped with the next [AnyEvent](https://metacpan.org/pod/AnyEvent) release. I hope this will happen
soon.
