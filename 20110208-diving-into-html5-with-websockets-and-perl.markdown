Title: Diving into HTML5 with WebSockets and Perl
Tags: Perl, HTML5, WebSockets
Comments: no

HTML5 is a new HTML standard that was adopted by W3C in 2007, and the first
public working draft was published in 2008. The work is still in progress, it
will reach Recommendation status probably in 2012, but many sections are already
stable and have usable implementations among various browsers.

[cut]

Among many new things it brings WebSockets. Websockets are bidirectional full
duplex persistent tcp connections between a server and a browser. In comparing to
other technics that are used nowadays like long polling, comet etc. it requires
only one connection, transmits only payload data (no http headers after
connection has established) and has an extremely simple asynchronous API on the
client side. Consider the following JavaScript code:

    var ws = new WebSocket('ws://localhost:3000');

    ws.onopen = function() {
        alert('Connection is established!');
    };

    ws.onclose = function() {
        alert('Connection is closed');
    };

    ws.onmessage = function(e) {
        var message = e.data;
        alert('Got new message: ' + message);
    };

    ws.send('Hello, world!');

Here we register callbacks (anonymous functions) that will be called when a
particular event occures. So `onopen` will be called when a connection with
WebSocket server is established, `onclose` when it is closed, `onmessage` when
a message is received. There is also `onerror` callback, but it is not shown
for simplicity. Every time when you want to send a message call `send`
function. As you can see, on the client side you don't have to worry about
handshakes, frames, buffering and so on.

Let's look at it on a deeper level.

WebSocket connection starts with a handshake between a server and a browser
using HTTP/1.1 `Upgrade` header. Here is an example what happens behind the
scene:

    > Browser request
    GET / HTTP/1.1
    Upgrade: WebSocket
    Connection: Upgrade
    Host: example.com
    Origin: http://example.com
    Sec-WebSocket-Key1: 18x 6]8vM;54 *(5:  {   U1]8  z [  8
    Sec-WebSocket-Key2: 1_ tx7X d  <  nw  334J702) 7]o}` 0

    Tm[K T2u

    < Server response
    HTTP/1.1 101 WebSocket Protocol Handshake
    Upgrade: WebSocket
    Connection: Upgrade
    Sec-WebSocket-Origin: http://example.com
    Sec-WebSocket-Location: ws://example.com/

    fQJ,fN/4F4!~K~MH

In the latest WebSocket specification draft #76 (which is now version 00
[http://tools.ietf.org/id/draft-ietf-hybi-thewebsocketprotocol-00](http://tools.ietf.org/id/draft-ietf-hybi-thewebsocketprotocol-00)) handshake
was changed adding `Sec-*` fields and challenges for various security reasons.
But there are still browsers that support draft #75 (e.g. some Android mobile
browsers) and current implementations should keep that in mind. Moreover the
protocol will change again (version 02
[http://tools.ietf.org/id/draft-ietf-hybi-thewebsocketprotocol-02](http://tools.ietf.org/id/draft-ietf-hybi-thewebsocketprotocol-02) already has
a much more complicated frame structure, current browsers don't support it
though), but the client API should stay the same.

Other `HTTP` headers passed along are mostly ignored, but there is one that is
definitely important, and that is `Cookie`. With use of cookies such important
tasks as authentication, user tracking and sessions can be performed.

After the handshake, if it was successful, the server and the browser can send
data back and forth using a simple framing (likely going to change):

    \x00...DATA...\xff

The data itself should be UTF-8 encoded. Specification also describes the ways
to transmit binary data, but it is not recommended for now and I will skip it
here too.

WebSocket connection could be secured using a standard TLS/SSL encription, the
client will be aware of this when connecting via `wss://` instead of `ws://`.

Using TLS/SSL is advised not only when you need to encrypt the traffic but also
when you need to bypass proxy servers and firewalls, that otherwise will just
block or not understand WebSockets connections.

## Browser Compatibility

As the WebSocket specifications change, it is worth knowing what browsers support
them. I've built a table of the clients that I checked:

    Draft #75
    ---------
    Chrome 5
    Safari 5.0

    Draft #76 (version 00)
    ---------
    Firefox 4b
    Safari  5.0.2
    Chrome  6
    Opera   10.70

    iOS     4.2

NOTE: Safari unfortunately does not support TLS/SSL WebSocket encryption.
Hopefully this will be changed soon.

NOTE2: Internet Explorer 9 will probably support WebSockets.

These are pretty new versions, that are not that popular nowadays. So does it
mean that we can't use WebSockets right now?

## Nonwebsocket Browser Support

Even if the most browsers will support WebSockets, there still will be those who
are either not updated or are not going to support them at all. For those some
really wise technics were implemented. I am going to talk about the two major
players, maybe there are others, but this is the whole another study.

### Flash Fallback

This workaround uses a Flash fallback that sets up a persistent connection and
does all the handshake parsing. So when a browser does not support WebSockets
natively it falls back to Flash.

There are two requirements that need to be set up for this to work. The first
one is to include script headers on your page that load Flash and appropriate
JavaScript code:

    <script type="text/javascript">
        // Only load the flash fallback when needed
        if (!('WebSocket' in window)) {
            document.write([
                '<scr'+'ipt type="text/javascript" '
                    + 'src="web-socket-js/swfobject.js"></scr'+'ipt>',
                '<scr'+'ipt type="text/javascript" '
                    + 'src="web-socket-js/FABridge.js"></scr'+'ipt>',
                '<scr'+'ipt type="text/javascript" '
                    + 'src="web-socket-js/web_socket.js"></scr'+'ipt>'
            ].join(''));
        }
    </script>
    <script type="text/javascript">
        if (WebSocket.__initialize) {
            // Set URL of your WebSocketMain.swf here:
            WebSocket.__swfLocation = 'web-socket-js/WebSocketMain.swf';
        }

        // Normal WebSocket initialization
    </script>

The second one is a Flash Socket Policy Server that must be run on port 843. All
it has to do is to send this message on every request:

    <?xml version="1.0"?>
    <!DOCTYPE cross-domain-policy SYSTEM
        "/xml/dtds/cross-domain-policy.dtd">
    <cross-domain-policy>
    <site-control permitted-cross-domain-policies="master-only"/>
    <allow-access-from domain="*" to-ports="*" secure="false"/>
    </cross-domain-policy>

There are some difficulties for users that are behind a proxy, but they could be
solved by manually passing proxy parameters to the WebSocket constructor or
letting the user specifing them.

I've personally used this workaround and it worked fine. Just don't forget to
start the Flash Policy Server or you will have a hard time searching for the
problems in your code.

### Socket.IO

This library is a unified class for many HTTP persistent connection technics
including not only WebSockets, but also Flash sockets, XHR long polling, iframes and
others.

It automatically guesses what transport to use depending on your browser. API is
close enough to WebSocket's.

    socket = new io.Socket('localhost');
    socket.connect();
    socket.on('connect', function(){
        // connected
    });
    socket.on('message', function(data){
        // data here
    });
    socket.send('some data');

To use it just include a single line in your html source file:

    <script src="http://cdn.socket.io/stable/socket.io.js"></script>

This library solves the proxy issue simply by switching to another transport.

While being a more advanced library it requires a smarter server that actually
supports all the transports too. You have to support not only WebSockets but
also HTTP/1.1 with all its complexity.

There is a Socket.IO for Perl implemented on top of [Mojolicious](https://metacpan.org/pod/Mojolicious)
[https://github.com/xantus/ignite](https://github.com/xantus/ignite) (long polling and WebSockets).

## WebSocket Issues

There are many reasons why WebSocket protocol will be likely changed again in
the future. Those issues are mostly provoked by the will of making them easy to
implement and to use.

### Reverse Proxy Support

The Draft #76 introduced a challenge that is sent as a body to the
WebSocket client. The problem is that it doesn't have a `Content-Length` header
and proxies will consider it as a next response or simply hang without any
error report to the user. Several solutions were proposed for fixing this issue
including adding a `Content-Length: 8` or `Connection: close` headers, using a
`POST` method and others.

### Framing

The `\x00..\xff` framing was considered not the best solution, since using just
one byte for the message separation should enough (versions 01, 02 introduce a
more advanced framing). It is also not clear if there are should be limitations
on frame sizes for preventing DDoS attacks for example.

### Other Issues

Other issues include: meta frames that could be used for additional settings,
like cookies setting after handshake; the needed level of HTTP compliancy;
importance of keep alive timeouts etc.

## Using WebSockets In A Real Life

What do we need to start using WebSockets? Of course we need a suitable web
browser. But that does not require any work. The real work is a WebSocket
server.

### WebSocket Server Requirements

The obvious requirements are:

- It must know how to parse WebSocket handshake and frames
- It must be asynchronous and nonblocking
- It must be able to hold many persistent connections

Parsing handshake and frames is not as hard as parsing HTTP/1.1 messages for
example, but does require some specification reading, including the fact that
they will probably change very soon.

Server must be asynchronous because of the WebSockets asynchronous nature. You
can receive message any time and you can send message any time, and at the same
time. Since threads are the root of all evils we will use nonblocking socket
reading and writing. The differences between asyncronous and nonblocking is
explained by only one phrase "Nonblocking mode tells us when to start the
action, asynchronous mode tells us when the action is finished".

The third requirement isn't really a requirement if you are going to use
WebSockets just for testing, but it is good to keep it mind.

### Perl Options

There are Perl Web Frameworks that support WebSockets out of the box, like
[Mojolicious](https://metacpan.org/pod/Mojolicious), which has a WebSocket client in addition, some work is being
done in [Dancer](https://metacpan.org/pod/Dancer), [Tatsumaki](https://metacpan.org/pod/Tatsumaki). Standalone WebSocket servers include those
built on top of various event loops, check out [Net::Async::WebSocket](https://metacpan.org/pod/Net::Async::WebSocket),
[AnyEvent::HTTP::Server](https://metacpan.org/pod/AnyEvent::HTTP::Server) and also [Plack](https://metacpan.org/pod/Plack) middleware
[https://github.com/motemen/Plack-Middleware-WebSocket](https://github.com/motemen/Plack-Middleware-WebSocket). More info on how to
build a WebSocket server in Perl you can find in the following article
[http://showmetheco.de/articles/2010/11/timtow-to-build-a-websocket-server-in-perl](http://showmetheco.de/articles/2010/11/timtow-to-build-a-websocket-server-in-perl).

The examples described later in this article use my own standalone WebSocket
server ReAnimator and event loop EventReactor, but they could be easily ported
to other event loops, since there is nothing special about parsing WebSocket
handshake (or by using [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket)) and doing nonblocking in Perl.

If you are considering what server to use check if it supports drafts #75 and
\#76 (version 00), has TLS/SSL support and understands Cookies. If you still
can't choose take one that has an asynchronous DNS resolution.

## WebSocket Demo Applications

### Configuration And Installation

Examples are accessible online from my GitHub page and usually don't require
third-party modules installation. The only exception is [JSON](https://metacpan.org/pod/JSON), but I am pretty
sure it is already installed on every Perl developer's machine.

### Simple Echo Server

In order to get started and get a feeling that something actually works, we are
going to try a simple chat server that is shipped with ReAnimator. If you clone
a git repository you will find `examples/` directory there (you can find more
examples of WebSocket echo servers in [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) distribution or on
GitHub [http://github.com/vti/protocol-websocket](http://github.com/vti/protocol-websocket) in ["" in examples](https://metacpan.org/pod/examples) directory).

    $ git clone http://github.com/vti/reanimator.git
    $ cd reanimator/

Now start the chat server (by default it will listen on 0.0.0.0:3000):

    $ perl examples/echo

And open a static `index.html` page, that lies in the `examples/public/`
directory, in your browser.

If you get a `Disconnected` message it means that either your browser does not
support WebSockets or there is a server problem. The server can be debugged by
setting up `EVENT_REACTOR_DEBUG=1` environmental variable.

On success you will get an `input` field and a `submit` button. If you send
something you should get it back. There is no fallback mechanism embedded, so
please use a browser that supports WebSockets natively.

Below is an explanation how everything actually works.

What's inside?

Inside we have a static `index.html` page that connects with a help of
JavaScript to the WebSocket server via API described earlier and there is an
`echo` Perl script with a few lines of code:

    use ReAnimator;

    my $server = ReAnimator->new;

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

This code should remind you the code in JavaScript, here we have an `on_accept`
hook that is run when a new connection is accepted, `on_message` hook that is
run when a new message (already correctly parsed and decoded to UTF-8) arrives.
A `send_message` method is used to send a message to the browser. In
this simple example we just send it back without any change.

To eliminate some confusions `on_accept` hook is called not when a normal
socket connection is accepted but after a WebSocket handshake. So you can be
sure that everything is prepared for bidirectional transmission. I didn't call
it `on_connect` because that would make a reader think that a server
connects to the browser, which is obviously the opposite.

The echo application can't demonstrate all the power of WebSockets of course, so
let's switch now to some real apps!

### Remote Shell

Of course the best way to administrate a server is via `ssh`. You get a full
featured terminal and you don't even notice that you are working on a remote
side. But sometimes you either don't have a `ssh` client installed on your
machine or there is no `ssh` access to your server.

The obvious solution is to use a browser, because it is installed everywhere and
you always can connect to any server. The problem is that to create a realtime
terminal in a browser some advanced techniques have to be used. Every time you
type a characted is should be sent immediately to the server. Every time a
server sends something back it should be displayed immediately too. And this is
asynchronous. HTML5 WebSockets is a perfect fit.

The example I am going to show is a true realtime terminal. The difference from
the most solutions is that a browser is just a thin layer between you and the
remote shell on the server. Browser doesn't send commands but characters and
special signals. The shell itself understands colors, terminal resizes, works
naturally with long running or interactive applications. Just as you would
expect a normal `ssh` client.

### Server Side

To implement this kind of interactivity we have to create a pseudo terminal on
server side to emulate a real terminal, parse special terminal characters like
colors, understand commands like `CTRL-C`, `CTRL-D` and so on.

For the pseudo terminal we will use [IO::Pty](https://metacpan.org/pod/IO::Pty), for the terminal emulation and
characters handling [Term::VT102](https://metacpan.org/pod/Term::VT102).

So to create a terminal the following steps should be made:

- Fork a child process
- Create a new pty object making it a slave to a normal tty
- Reopen `STDIN`, `STDOUT` and `STDERR` so they point to our pty object
- Exec a command like `/bin/sh` to start a shell inside a new terminal
- Register callbacks that are invoked when a new data arrives

This is all for the server side. For simplicity I put everything in one class
and called it `Terminal.pm`, you can find the source code on GitHub
[https://github.com/vti/showmetheshell/blob/master/lib/Terminal.pm](https://github.com/vti/showmetheshell/blob/master/lib/Terminal.pm). Now
we just register descriptors in our event loop. In ReAnimator it could look like
this (simplified):

    my $server = ReAnimator->new;

    ReAnimator->new(
        on_accept => sub {
            my ($self, $client) = @_;

            my $terminal = Terminal->new(
                cmd            => '/bin/sh',
                on_row_changed => sub {
                    my ($self, $row, $text) = @_;

                    $client->send_message(
                        JSON->new->encode(
                            {type => 'row', row => $row, text => $text}
                        )
                    );
                }
            );

            $self->event_reactor->add_atom($terminal);

            $terminal->start;

            $client->on_message(
                sub {
                    my ($client, $message) = @_;

                    ...
                    # Send a pressed key to the terminal
                    $terminal->key($code);
                }
            );
        }
    )->listen->start;

And that's it. Every time something happens in a terminal it is sent to the
browser and when something is typed in the browser it is sent to the terminal.

### Client Side

The biggest problem on the client side it to catch all the key presses. It is
even more difficult when it comes to different browsers. But using a modern js
framework this can be greatly simplified (e.g. `JQuery`). Another solution is
to create a virtual keyboard.

Since the server sends which row has changed, there are no issues displaying it
with `JavaScript`. I've created 24 `span` rows with unique ids and used this
function:

    function(n, data) {
        var row = $('#row' + n);
        row.html(data);
    };

Seriously, this is all that is needed. And WebSocket interaction stays the same.

### Important Enhancement

The obvious enhancement is to use SSL of course. You don't want to run a `ssh`
session via a not protected connection. And since WebSockets support SSL it's as
easy as replacing `ws://` with `wss://` (evidently, the server must support
SSL too).

### Demo And Source code

The full source code of this example is available at
[http://github.com/vti/showmetheshell](http://github.com/vti/showmetheshell). The demo video is also available at
[http://vimeo.com/13681309](http://vimeo.com/13681309).

### VNC

If you ever wanted to connect to other PC and be able actually see the desktop
and use your keyboard and mouse (not via `ssh`) you probably know what VNC is.
VNC is a really true thin client, that uses RFB (Remote FrameBuffer) protocol.
It has a client-server architecture. All the display pixels computation is done
on the server side and client only requests framebuffer updates. Data between
client and server is transfered using various encodings to minimize traffic.
Redrawing and input control can be really responsive.

There are many VNC servers and clients, some of the operating systems even have
a VNC server out of the box. For example so does Mac OS with its Screen sharing
settings. Most of the Linux distributions provide several alternatives that
could be installed with one command. There are also open source variants for
Windows users too.

But what to do if you have only a browser and internet connection? There are
java applets that come with some VNC servers that you can use for accessing a
remote machine. But that requires Java support, which sometimes could be a
problem. Anyway we want a native solution. And that solution is WebSockets. They
are ideal for persistent connections, they can transmit data without overhead
and they are fun and easy to use.

To be able to use RFB protocol we must know how to understand it and display it
in a browser. While RFB operates on pixels we can use HTML5 `canvas` that has an
extremely easy API to draw images, copy rectangles and more. Consider the
following example:

    var canvas = document.getElementById('canvas');
    var context = canvas.getContext('2d');

    // Draw image
    context.drawImage(img, x, y);

    // Copy rectangle
    var img = context.getImageData(x, y, width, height);
    context.putImageData(img, x, y);

We can proxy RFB message to the browser without any modification but then we
have to parse them using JavaScript which is not a good idea, since not only
that could take awhile, but it is not that easy to work with binary data in
JavaScript. We can parse RFB message on the server side, tranform that to some
kind of a textual form and deliver it to the browser.

All the user input interaction like a keyboard button pressing or mouse moves
can be captured using JavaScript without any problems.

When it comes to Perl and CPAN the only module that works with VNC is Net::VNC,
but it only captures a screenshot and is not useful for other things. That is
why I had to write a Protocol::RFB module, that understands most of the RFB
messages, can deal with keyboard and mouse, provides hooks that can be used in
asynchronous applications.

But now the WebSocket server must not only listen to the browser's connection
but it must also listen to the VNC server on another address. It has to have a
simple mechanism to pass messages from the client to the server and vice versa.

Using ReAnimator and EventReactor it is very straightforward (code is
simplified):

    ReAnimator->new(
        on_accept => sub {
            my ($self, $client) = @_;

            my $vnc = Protocol::RFB::Client->new(password => $VNC_PASSWORD);

            my $slave = $self->event_reactor->connect(
                address    => $VNC_ADDRESS,
                port       => $VNC_PORT,
                on_connect => sub {
                    my $slave = shift;

                    $slave->on_read(
                        sub {
                            my $slave = shift;
                            my $chunk = shift;

                            $vnc->parse($chunk);
                        }
                    );

                    $slave->on_disconnect(sub { $self->drop($client) });
                }
            );

            $vnc->on_handshake(
                sub {
                    my $vnc = shift;

                    $client->send_message(
                        JSON->new->encode(
                            {   type   => 's',
                                name   => $vnc->server_name,
                                width  => $WIDTH || $vnc->width,
                                height => $HEIGHT || $vnc->height
                            }
                        )
                    );
                }
            );

            $vnc->on_framebuffer_update(
                sub {
                    my ($vnc, $message) = @_;

                    foreach my $rectangle (@{$message->rectangles}) {
                        $client->send_message(
                            JSON->new->encode(
                                {type => 'fu', rectangle => $rectangle}
                            )
                        );
                    }
                }
            );

            $vnc->on_write(
                sub {
                    my ($vnc, $chunk) = @_;

                    $slave->write($chunk);
                }
            );

            $client->on_message(
                sub {
                    my ($client, $message) = @_;

                    my $json = JSON->new;

                    eval { $message = $json->decode($message); };
                    return if !$message || $@;

                    if ($message->{type} eq 'fuq') {
                        ...
                    }

                    ...
                }
            );
        }
    )->listen->start;

Here the WebSocket and VNC connections run in the same event loop, they just
have a different level of abstraction. While the WebSocket connection operates
on text messages, VNC connection operates on binary data.

The actual speed of rendering is acceptable. The bottle neck is obviously Perl
itself. That could be optimised writing an XS version, but it's more than enough
for demonstating purposes.

### Demo And Source Code

The full source code of this example is available at
[http://github.com/vti/showmethedesktop](http://github.com/vti/showmethedesktop). To run this application you need a
second machine with a VNC server, this could be for example a Linux distibution
running on a virtual machine. The demo video is also available at
[http://vimeo.com/16459612](http://vimeo.com/16459612).

## Conclusion

HTML5 WebSockets is a powerful way for creating realtime applications that
require a persistent connection and bidirectional communication. The future of
the Web is here and Perl is ready to accept this challenge.

    This article was written for the $foo German Perl Magazine. So if you can read
    German and want to support a good Perl periodical publication check out
    http://www.perl-magazin.de/!
