Title: Using Protocol::WebSocket with Plack
Tags: Perl, plack, websocket

I recently added [PSGI](https://metacpan.org/pod/PSGI) support to [Protocol::Websocket](https://metacpan.org/pod/Protocol::Websocket). And below is an
example of WebSocket echo server using [Plack](https://metacpan.org/pod/Plack), [AnyEvent](https://metacpan.org/pod/AnyEvent) and [Twiggy](https://metacpan.org/pod/Twiggy).

[cut]

    #!/usr/bin/env perl

    use strict;
    use warnings;

    use AnyEvent::Handle;
    use Protocol::WebSocket::Handshake::Server;
    use Protocol::WebSocket::Frame;

    my $psgi_app = sub {
        my $env = shift;

        my $fh = $env->{'psgix.io'} or return [500, [], []];

        my $hs = Protocol::WebSocket::Handshake::Server->new_from_psgi($env);
        $hs->parse($fh) or return [400, [], [$hs->error]];

        return sub {
            my $respond = shift;

            my $h = AnyEvent::Handle->new(fh => $fh);
            my $frame = Protocol::WebSocket::Frame->new;

            $h->push_write($hs->to_string);

            $h->on_read(
                sub {
                    $frame->append($_[0]->rbuf);

                    while (my $message = $frame->next) {
                        $message = Protocol::WebSocket::Frame->new($message)->to_string;
                        $h->push_write($message);
                    }
                }
            );
        };
    };

    $psgi_app;

I hope the code is pretty much straightforward. With the help of
[Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) we get `75` and `76` support, easy frame parsing and
nice protocol abstraction.

You try it just by running

    $ twiggy app.psgi
