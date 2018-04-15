Title: TIMTOW to build a WebSocket server in Perl
Tags: Perl, WebSockets, event loop
Comments: no

Today I am going to describe many ways how to build a WebSocket server in
Perl, including ready to go WebSocket servers and web frameworks with no
additional hacking, async servers and web frameworks that can be easily extended
and low level ways to create a WebSocket server with a help of
[Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) module and many available event loops, and even a simple
echo server using only [IO::Socket::INET](https://metacpan.org/pod/IO::Socket::INET) and [IO::Poll](https://metacpan.org/pod/IO::Poll).

[cut]

Last release date and version is provided for every module that I tried. If you
know any other ways (I am sure there are) to build a WebSocket server in Perl,
please, let me know and I will include it in this article.

The order of appearance is due to the last CPAN release date or last repository
commit date if the distribution is not yet on CPAN.

All the examples are runnable and hopefully will work if you try them too. All
the examples are echo WebSocket servers. Error handling is omitted for
simplicity.

## Ready to go WebSocket servers and Web Frameworks

The following list contains servers and web frameworks that don't need any
additional hacking. They can be used out of the box.

- Nov 2010 ReAnimator 0.0001

    This is a server that I wrote, sorry for it being on the top of the list :). It
    supports 75 and 76 drafts, has SSL/TLS support and HTTP/1.1 cookies. It also has
    a WebSocket client.

        # reanimator.pl
        use ReAnimator;

        ReAnimator->new(
            on_accept => sub {
                my ($self, $client) = @_; 

                $client->on_message(
                    sub {
                        my ($client, $message) = @_; 

                        $client->send_message($message);
                    }
                );
            }
        )->listen->start;

        $ perl reanimator.pl

    ReAnimator is not available on CPAN yet, you can find the source code on GitHub
    [http://github.com/vti/reanimator](http://github.com/vti/reanimator).

- Nov 2010 [Mojolicious](https://metacpan.org/pod/Mojolicious) 0.999941

    This is a great web framework that has WebSockets out of the box. It supports 76
    draft, SSL/TLS, full-stack HTTP/1.1 and async DNS resolution.

        # mojolicious.pl
        use Mojolicious::Lite;

        websocket '/' => sub {
            my $self = shift;

            $self->on_message(
                sub {
                    my ($self, $message) = @_;

                    $self->send_message($message);
                }
            );
        };

        app->start;

        $ perl mojolicious.pl

    Or even with one-liner:

        $ perl -Mojo -e 'a("/" => sub { shift->on_message( \
            sub { shift->send_message("$_[0]") }) })->start' daemon

- Nov 2010 [Dancer](https://metacpan.org/pod/Dancer) 1.2000

    [Dancer](https://metacpan.org/pod/Dancer)'s WebSocket support is experimental, but there is a chat plugin
    available on Github [https://github.com/franckcuny/dancer-chat](https://github.com/franckcuny/dancer-chat). It uses
    [Web::Hippie](https://metacpan.org/pod/Web::Hippie) middleware.

- Oct 2010 [Plack::Middleware::WebSocket](https://metacpan.org/pod/Plack::Middleware::WebSocket)

    This [Plack](https://metacpan.org/pod/Plack) middleware is not CPAN, but available from GitHub
    [https://github.com/motemen/Plack-Middleware-WebSocket](https://github.com/motemen/Plack-Middleware-WebSocket). Below is a simplified
    example from the distribution:

        # app.psgi
        use Plack::Builder;
        use Plack::Request;
        use AnyEvent;
        use AnyEvent::Handle;
        
        my $app = sub {
            my $env = shift;
            my $req = Plack::Request->new($env);
            my $res = $req->new_response(200);
        
            if (my $fh = $env->{'websocket.impl'}->handshake) {
                return start_ws_echo($fh);
            }
            $res->code($env->{'websocket.impl'}->error_code);
        
            return $res->finalize;
        };
        
        sub start_ws_echo {
            my ($fh) = @_;
        
            my $handle = AnyEvent::Handle->new(fh => $fh);
            return sub {
                my $respond = shift;
        
                on_read $handle sub {
                    shift->push_read(
                        'AnyEvent::Handle::Message::WebSocket',
                        sub {
                            my $msg = $_[1];
                            my $w;
                            $w = AE::timer 1, 0, sub {
                                $handle->push_write(
                                    'AnyEvent::Handle::Message::WebSocket', $msg);
                                undef $w;
                            };
                        },
                    );
                };
            };
        }
        
        builder {
            enable 'WebSocket';
            $app;
        };

        $ plackup app.psgi --port 3000

- Jun 2010 AnyEvent::HTTP::Server [https://github.com/Mons/AnyEvent-HTTP-Server](https://github.com/Mons/AnyEvent-HTTP-Server)

    This server supports drafts 75 and 76 and provides all the power that
    [AnyEvent](https://metacpan.org/pod/AnyEvent) has, including SSL, async DNS etc.

    Some author's modules are required to run this application, like [uni::perl](https://metacpan.org/pod/uni::perl),
    [accessors::fast](https://metacpan.org/pod/accessors::fast), `HTTP::Easy` [https://github.com/Mons/HTTP-Easy](https://github.com/Mons/HTTP-Easy).

    Example below is greatly simplified. Full source can be found in repository
    [https://github.com/Mons/AnyEvent-HTTP-Server/blob/master/ex/ws-echo/server.pl](https://github.com/Mons/AnyEvent-HTTP-Server/blob/master/ex/ws-echo/server.pl).

        # anyevent-http-server.pl
        use AnyEvent::Impl::Perl;
        use AE;
        use AnyEvent::HTTP::Server;

        AnyEvent::HTTP::Server->new(
            host    => '0.0.0.0',
            port    => 3000,
            pid     => '/tmp/wsecho.pid',
            request => sub {
                my $r = shift;

                $r->upgrade(
                    websocket => sub {
                        my $ws = shift;

                        $ws->onmessage(sub { $ws->send(@_) });
                    }
                );

                return 1;
            }
        )->start;

        AE::cv->recv;

        $ perl anyevent-http-server.pl

## Extendable async servers and web frameworks

- Oct 2010 [Twiggy](https://metacpan.org/pod/Twiggy) 0.1008

    I coudn't make draft 76 to work with [Twiggy](https://metacpan.org/pod/Twiggy), but there is a draft 75 WebSocket
    example in the distribution
    [http://cpansearch.perl.org/src/MIYAGAWA/Twiggy-0.1008/eg/chat-websocket/chat.psgi](http://cpansearch.perl.org/src/MIYAGAWA/Twiggy-0.1008/eg/chat-websocket/chat.psgi).

- Apr 2010 [Tatsumaki](https://metacpan.org/pod/Tatsumaki) 0.1010

    There is a development branch called `websocket`, and there is an example of
    websocket chat, but unfortunately it supports only draft 75.

    You can check it out on GitHub
    [https://github.com/miyagawa/Tatsumaki/commit/38fa7f5d215dd683f95949cc8fdad8565a13b954](https://github.com/miyagawa/Tatsumaki/commit/38fa7f5d215dd683f95949cc8fdad8565a13b954).

Actually theoretically any async web framework running on [Plack](https://metacpan.org/pod/Plack) with
`streaming` and `nonblocking` support can be a WebSocket server. But
practically I coudn't find one.

## Extendable with [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) low level tcp event loops

First of all what [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) is? This is a set of modules for
parsing WebSocket messages and constructing them. It doesn't provide any server
or anything like that. Just parsing and constructing. This way we can use it in
any environment.

You can read more about [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) in my previous post
[http://showmetheco.de/articles/2010/11/parsing-websocket-messages-with-protocol-websocket.html](http://showmetheco.de/articles/2010/11/parsing-websocket-messages-with-protocol-websocket.html).

- Nov 2010 [EventReactor](https://metacpan.org/pod/EventReactor) 0.0001

    ReAnimator itself uses [EventReactor](https://metacpan.org/pod/EventReactor) as event loop, which obviously makes it
    perfect for WebSockets too.

        # event_reactor.pl
        use EventReactor;

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        EventReactor->new(
            address   => 'localhost',
            port      => 3000,
            on_accept => sub {
                my ($self, $client) = @_;

                my $hs = Protocol::WebSocket::Handshake::Server->new;
                my $frame = Protocol::WebSocket::Frame->new;

                $client->on_read(
                    sub {
                        my ($client, $chunk) = @_;

                        if (!$hs->is_done) {
                            $hs->parse($chunk);

                            if ($hs->is_done) {
                                $client->write($hs->to_string);
                            }

                            return;
                        }

                        $frame->append($chunk);

                        while (my $message = $frame->next) {
                            $client->write($frame->new($message)->to_string);
                        }
                    }
                );
            }
        )->listen->start;

        $ perl event_reactor.pl

    EventReactor is not available on CPAN yet, you can find the source code on GitHub
    [http://github.com/vti/event\_reactor](http://github.com/vti/event_reactor).

- Nov 2010 [POE::Component::Server::TCP](https://metacpan.org/pod/POE::Component::Server::TCP) 1.294

    [POE](https://metacpan.org/pod/POE) is probably the most well known event loop for Perl. It has a huge
    collection of modules for any needs. This implementation uses
    [POE::Component::Server::TCP](https://metacpan.org/pod/POE::Component::Server::TCP) which provides a simple `tcp` server.

        # poe.pl
        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        use POE qw(Component::Server::TCP);

        my $hs    = Protocol::WebSocket::Handshake::Server->new;
        my $frame = Protocol::WebSocket::Frame->new;

        POE::Component::Server::TCP->new(
            Port         => 3000,
            ClientFilter => 'POE::Filter::Stream',
            ClientInput  => sub {
                my $chunk = $_[ARG0];

                if (!$hs->is_done) {
                    $hs->parse($chunk);

                    if ($hs->is_done) {
                        $_[HEAP]{client}->put($hs->to_string);
                    }

                    return;
                }

                $frame->append($chunk);

                while (my $message = $frame->next) {
                    $_[HEAP]{client}->put($frame->new($message)->to_string);
                }
            }
        );

        POE::Kernel->run;

        $ perl poe.pl

- Nov 2010 [IO::Async](https://metacpan.org/pod/IO::Async) 0.31

    I really liked working with this module, maybe because it looks like
    EventReactor that I wrote.

        # io-async.pl
        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        use IO::Socket::INET;
        use IO::Async::Listener;

        use IO::Async::Loop;
        my $loop = IO::Async::Loop->new;

        my $listener = IO::Async::Listener->new(
            on_stream => sub {
                my ($self, $stream) = @_;

                my $hs    = Protocol::WebSocket::Handshake::Server->new;
                my $frame = Protocol::WebSocket::Frame->new;

                $stream->configure(
                    on_read => sub {
                        my ($self, $buffref, $closed) = @_;

                        if (!$hs->is_done) {
                            $hs->parse($$buffref);

                            if ($hs->is_done) {
                                $self->write($hs->to_string);
                            }

                            $$buffref = "";
                            return 0;
                        }

                        $frame->append($$buffref);

                        while (my $message = $frame->next) {
                            $self->write($frame->new($message)->to_string);
                        }

                        $$buffref = "";
                        return 0;
                    }
                );

                $loop->add($stream);
            }
        );

        $loop->add($listener);

        my $socket = IO::Socket::INET->new(
            LocalAddr => 'localhost',
            LocalPort => 3000,
            Listen    => 1,
        );

        $listener->listen(handle => $socket);

        $loop->loop_forever;

        $ perl io-async.pl

- Oct 2010 [AnyEvent](https://metacpan.org/pod/AnyEvent) 5.28

    [AnyEvent](https://metacpan.org/pod/AnyEvent) is probably the most popular event loop in Perl now, because it
    allows you actually run on any other low level loop or multiplexer, hiding all
    their diversities and complexities.

        # anyevent.pl
        use AnyEvent::Socket;
        use AnyEvent::Handle;

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        my $cv = AnyEvent->condvar;

        my $hdl;

        AnyEvent::Socket::tcp_server undef, 3000, sub {
            my ($clsock, $host, $port) = @_;

            my $hs    = Protocol::WebSocket::Handshake::Server->new;
            my $frame = Protocol::WebSocket::Frame->new;

            $hdl = AnyEvent::Handle->new(fh => $clsock);

            $hdl->on_read(
                sub {
                    my $hdl = shift;

                    my $chunk = $hdl->{rbuf};
                    $hdl->{rbuf} = undef;

                    if (!$hs->is_done) {
                        $hs->parse($chunk);

                        if ($hs->is_done) {
                            $hdl->push_write($hs->to_string);
                            return;
                        }
                    }

                    $frame->append($chunk);

                    while (my $message = $frame->next) {
                        $hdl->push_write($frame->new($message)->to_string);
                    }
                }
            );
        };

        $cv->wait;

        $ perl anyevent.pl

    Other nice chat example built on top of [AnyEvent](https://metacpan.org/pod/AnyEvent) you can find on GitHub
    [https://github.com/typester/anyevent-websocket-demo/blob/master/chat.pl](https://github.com/typester/anyevent-websocket-demo/blob/master/chat.pl).

- Oct 2010 [Reflex](https://metacpan.org/pod/Reflex) 0.085

    Object oriented events using [Moose](https://metacpan.org/pod/Moose) and [POE](https://metacpan.org/pod/POE) to support many event loops and many syntaxes.

        # reflex.pl
        {
            package EchoStream;
            use Moose;
            extends 'Reflex::Stream';
            use Protocol::WebSocket::Handshake::Server;
            use Protocol::WebSocket::Frame;
        
            has hs => (
                is      => 'ro',
                isa     => 'Protocol::WebSocket::Handshake::Server',
                default => sub { Protocol::WebSocket::Handshake::Server->new() },
            );
        
            has frame => (
                is      => 'ro',
                isa     => 'Protocol::WebSocket::Frame',
                default => sub { Protocol::WebSocket::Frame->new() },
            );
        
            sub on_data {
                my ($self, $args) = @_;
        
                my $hs = $self->hs;
                unless ($hs->is_done) {
                    $hs->parse($args->{data});
                    $self->put($hs->to_string) if $hs->is_done;
                    return;
                }
        
                my $frame = $self->frame;
                $frame->append($args->{data});
                while (my $message = $frame->next) {
                    $self->put($frame->new($message)->to_string);
                }
            }
        }
        
        {
            package TcpEchoServer;
        
            use Moose;
            extends 'Reflex::Acceptor';
            use Reflex::Collection;
        
            has_many clients => (handles => {remember_client => "remember"});
        
            sub on_accept {
                my ($self, $args) = @_;
                $self->remember_client(
                    EchoStream->new(handle => $args->{socket}, rd => 1));
            }
        }
        
        TcpEchoServer->new(
            listener => IO::Socket::INET->new(
                LocalAddr => '127.0.0.1',
                LocalPort => 3000,
                Listen    => 5,
                Reuse     => 1,
            )
        )->run_all;

        $ perl reflex.pl

- Jul 2010 [Net::Server::Multiplex](https://metacpan.org/pod/Net::Server::Multiplex) 0.99

    [Net::Server::Multiplex](https://metacpan.org/pod/Net::Server::Multiplex) is like [IO::Multiplex](https://metacpan.org/pod/IO::Multiplex), but has a more high-level
    API (not that much higher though).

        # net-server-multiplex.pl
        package SampleChatServer;

        use strict;
        use warnings;

        use base 'Net::Server::Multiplex';

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        __PACKAGE__->run;

        my $hs;
        my $frame;

        sub mux_connection {
            my $self = shift;
            my ($mux, $fh) = @_;
            my $peer = $self->{peeraddr};

            $self->{id}       = $self->{net_server}->{server}->{requests};
            $self->{peerport} = $self->{net_server}->{server}->{peerport};
        }

        sub mux_input {
            my $self = shift;
            my ($mux, $fh, $in_ref) = @_;

            $hs    ||= Protocol::WebSocket::Handshake::Server->new;
            $frame ||= Protocol::WebSocket::Frame->new;

            if (!$hs->is_done) {
                $hs->parse($$in_ref);

                if ($hs->is_done) {
                    print $fh $hs->to_string;
                }

                $$in_ref = "";
                return 0;
            }

            $frame->append($$in_ref);

            while (my $message = $frame->next) {
                print $fh $frame->new($message)->to_string;
            }

            $$in_ref = "";
        }

        $ perl net-server-multiplex.pl --port 3000

- Apr 2010 [IO::Lambda](https://metacpan.org/pod/IO::Lambda)

    This is a really unique kind of event loop. It uses its own syntax and sometimes
    you have to wrap your head around it. While trying to implement a WebSocket echo
    server I had a few moments thinking that I am not gonna make it, but I did and
    here it is:

        # io-lambda.pl
        use IO::Socket;
        use IO::Lambda qw(:all);
        use IO::Lambda::Socket qw(:all);

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        my $conn_timeout = 10;

        my $server = IO::Socket::INET->new(
            Listen    => 5,
            LocalPort => 3000,
            Blocking  => 0,
            ReuseAddr => 1,
        ) or die $!;

        my $serv = lambda {
            context $server;
            accept {
                my $conn = shift;

                again;

                $conn->blocking(0);

                my $hs    = Protocol::WebSocket::Handshake::Server->new;
                my $frame = Protocol::WebSocket::Frame->new;

                my $buf = '';
                context readbuf, $conn, \$buf, qr/^(.*)$/s, $conn_timeout;

            tail {
                my ($match, $error) = @_;

                return close($conn) unless defined $match;

                substr($buf, 0, length($match)) = '';

                my $res = '';
                if (!$hs->is_done) {
                    $hs->parse($match);

                    if ($hs->is_done) {
                        $res = $hs->to_string;
                    }

                    $match = '';
                }

                $frame->append($match);

                while (my $message = $frame->next) {
                    $res .= $frame->new($message)->to_string;
                }

                again;

                $match = '';
                context writebuf, $conn, \$res, length($res), 0, $conn_timeout;
                &tail();
            }}
        };

        $serv->wait;

        $ perl io-lambda.pl

- Jul 2009 [IO::Event](https://metacpan.org/pod/IO::Event) 0.704

        # io-event.pl
        use IO::Event;

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        my $ioe = IO::Event::Socket::INET->new(
            LocalAddr => 'localhost',
            LocalPort => 3000,
            Listen    => 1,
            Blocking  => 0
        );

        IO::Event::loop;

        my $hs;
        my $frame;

        sub ie_connection {
            my ($handler, $ioe) = @_;

            $hs    = Protocol::WebSocket::Handshake::Server->new;
            $frame = Protocol::WebSocket::Frame->new;

            $ioe->accept;
        }

        sub ie_input {
            my ($handler, $client, $input_buffer_reference) = @_;

            if (!$hs->is_done) {
                $hs->parse($$input_buffer_reference);

                if ($hs->is_done) {
                    print $client $hs->to_string;
                }

                $$input_buffer_reference = '';
                return;
            }

            $frame->append($$input_buffer_reference);

            while (my $message = $frame->next) {
                print $client $frame->new($message)->to_string;
            }

            $$input_buffer_reference = '';
            return;
        }

        $ perl io-event.pl

- Sep 2008 [IO::Multiplex](https://metacpan.org/pod/IO::Multiplex) 1.10

        use IO::Socket;
        use IO::Multiplex;

        use Protocol::WebSocket::Handshake::Server;
        use Protocol::WebSocket::Frame;

        my $mux = new IO::Multiplex;

        my $sock = new IO::Socket::INET(
            Proto     => 'tcp',
            LocalPort => 3000,
            Listen    => 1
        ) or die "socket: $@";

        $mux->listen($sock);

        $mux->set_callback_object(__PACKAGE__);
        $mux->loop;

        my $hs;
        my $frame;

        sub mux_input {
            my $package = shift;
            my $mux     = shift;
            my $fh      = shift;
            my $input   = shift;

            $hs    ||= Protocol::WebSocket::Handshake::Server->new;
            $frame ||= Protocol::WebSocket::Frame->new;

            foreach my $c ($mux->handles) {
                if (!$hs->is_done) {
                    $hs->parse($$input);

                    if ($hs->is_done) {
                        print $c $hs->to_string;
                    }

                    $$input = '';
                    return;
                }

                $frame->append($$input);

                while (my $message = $frame->next) {
                    print $c $frame->new($message)->to_string;
                }
            }

            $$input = '';
        }

- Mar 2003 [Net::Socket::NonBlock](https://metacpan.org/pod/Net::Socket::NonBlock) 0.15

    I wanted to use this module to write a WebSocket server, but after looking at
    the tests and seeing this:

        ok(1); # If we made it this far, we're ok.

    I closed it and never want to open it again.

## Using just [IO::Socket::INET](https://metacpan.org/pod/IO::Socket::INET) and [IO::Poll](https://metacpan.org/pod/IO::Poll)

A real low level stuff for creating a WebSocket server in Perl. We use
[IO::Poll](https://metacpan.org/pod/IO::Poll) for multiplexing, but any other multiplexer can be used of course.
We accept only one connection, echo all the messages and exit when a client
disconnects.

    # server.pl
    use IO::Socket::INET;
    use IO::Poll qw/POLLIN/;

    use Protocol::WebSocket::Handshake::Server;
    use Protocol::WebSocket::Frame;

    my $poll = IO::Poll->new;

    my $socket = IO::Socket::INET->new(
        Blocking  => 0,
        LocalAddr => 'localhost',
        LocalPort => 3000,
        Proto     => 'tcp',
        Type      => SOCK_STREAM,
        Listen    => 1
    );

    $socket->blocking(0);

    $socket->listen;

    my $client;

    while (1) {
        if ($client = $socket->accept) {
            $poll->mask($client => POLLIN);
            last;
        }
    }

    my $hs    = Protocol::WebSocket::Handshake::Server->new;
    my $frame = Protocol::WebSocket::Frame->new;

    LOOP: while (1) {
        $poll->poll(0.1);

        foreach my $reader ($poll->handles(POLLIN)) {
            my $rs = $client->sysread(my $chunk, 1024);
            last LOOP unless $rs;

            if (!$hs->is_done) {
                $hs->parse($chunk);

                if ($hs->is_done) {
                    $client->syswrite($hs->to_string);
                }

                next;
            }

            $frame->append($chunk);

            while (my $message = $frame->next) {
                $client->syswrite($frame->new($message)->to_string);
            }
        }
    }

    $ perl server.pl

## Conclusion

TIMTOWTDI!
