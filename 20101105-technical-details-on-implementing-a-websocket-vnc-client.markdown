Title: Technical details on implementing a WebSocket VNC client
Tags: Perl, WebSockets, VNC
Comments: no

Now I am going to explain more details about the previous
[http://showmetheco.de/articles/2010/11/perl-websockets-and-vnc-in-your-browser.html](http://showmetheco.de/articles/2010/11/perl-websockets-and-vnc-in-your-browser.html)
post.

[cut]

If you are familiar with a VNC model, you know that it uses an RFB protocol and
there are lot of desktop clients and servers (btw Mac OS has it under `Screen
sharing` settings).

When it comes to Perl and CPAN the only module that works with VNC is
[Net::VNC](https://metacpan.org/pod/Net::VNC), but it only captures a screenshot and is not useful for other
things. That is why I had to write `Protocol::RFB`
[http://github.com/vti/protocol-rfb](http://github.com/vti/protocol-rfb). It is not finished but can parse many
RFB messages including keyboard and mouse interactions. Some day it will be
released on CPAN and there is going to be another module when you search for
VNC, hopefully.

Another issue is a WebSocket server. In a previous version of
`showmethedesktop` I used [Mojolicious](https://metacpan.org/pod/Mojolicious) for WebSockets and JSON parsing, but
WebSockets is still a work in progress and [Mojolicious](https://metacpan.org/pod/Mojolicious) does not support all
the protocol drafts. Moreover I needed a mechanism to connect WebSockets to
normal sockets. It can be done using [Mojo::IOLoop](https://metacpan.org/pod/Mojo::IOLoop) but requires a bit of low
level work. So I've came up with `reanimator`
[http://github.com/vti/reanimator](http://github.com/vti/reanimator) that has all the features I need for my
WebSocket applications.

Consider this simplified example of `showmethedesktop` code:

        my $server = ReAnimator->new;

        $server->on_connect(
            sub {
                my ($self, $client) = @_;

                my $vnc = Protocol::RFB::Client
                        ->new(password => 'vnc password');

                my $slave = $self->connect(
                    address => 'localhost',
                    port    => '5901',
                    on_read => sub {
                        my $slave = shift;
                        my $chunk = shift;

                        $vnc->parse($chunk);
                    }
                );

                $vnc->on_write(
                    sub {
                        my ($vnc, $chunk) = @_;

                        $slave->write($chunk);
                    }
                );

                $vnc->on_framebuffer_update(
                    sub {
                        my ($vnc, $message) = @_;
         
                        $client->send_message(...);
                    }
                );

                $client->on_message(
                    sub {
                        my ($client, $message) = @_;

                        ...
                        $vnc->pointer_event($message->{x},
                                $message->{y}, $mask);
                        ...
                    }
                );
            }
        );

        $server->listen;

Creating a slave connection that is tied to a WebSocket is realy easy. You just
register appropriate callbacks and Perl subroutines will save the context.
Every time when something comes from a browser it is passed to vnc, and vice
versa without need to control connections by yourself.

Now about the client side. The drawing is done using an HTML5 canvas. It is
fairly easy to create your own images, fill in pixels or copy rectangles.
Capturing keyboard and mouse events is not harder when you use JQuery.

This might look like a really simple task, and... it is! WebSockets is fun. If
you want more details you can always check the source code
[http://github.com/vti/showmethedesktop](http://github.com/vti/showmethedesktop) or write me a message, or even better
a comment to this post.

Thanks for reading!
