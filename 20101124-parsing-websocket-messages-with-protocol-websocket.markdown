Title: Parsing WebSocket messages with Protocol::WebSocket
Tags: Perl, WebSocket
Comments: no

There are quite a few modules on CPAN that parse HTTP messages. But there is no
module that parses WebSockets messages. I mean not a server or a web framework
that supports WebSockets, but a separate module that does only one thing.
[Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) is a module that is able to parse and construct WebSocket
client/server handshakes and frames, and does only this.

[cut]

It is still a work in progress, but it already supports drafts 75 and 76,
websocket urls, cookies and has a simple API.

All the classes obviously have `parse` and `to_string` methods that parse and
construct messages respectively.

## Parsing and constructing a server handshake

    my $h = Protocol::WebSocket::Handshake::Server->new;

    # Parse client request
    $h->parse(<<"EOF");
    GET /demo HTTP/1.1
    Upgrade: WebSocket
    Connection: Upgrade
    Host: example.com
    Origin: http://example.com
    Sec-WebSocket-Key1: 18x 6]8vM;54 *(5:  {   U1]8  z [  8
    Sec-WebSocket-Key2: 1_ tx7X d  <  nw  334J702) 7]o}` 0

    Tm[K T2u
    EOF

    $h->error;   # Check if there were any errors
    $h->is_done; # Returns 1

    # Create response
    $h->to_string; # HTTP/1.1 101 WebSocket Protocol Handshake
                   # Upgrade: WebSocket
                   # Connection: Upgrade
                   # Sec-WebSocket-Origin: http://example.com
                   # Sec-WebSocket-Location: ws://example.com/demo
                   #
                   # fQJ,fN/4F4!~K~MH

## Parsing and constructing a client handshake

    my $h = Protocol::WebSocket::Handshake::Client->new;

    # Create request
    $h->to_string; # GET /demo HTTP/1.1
                   # Upgrade: WebSocket
                   # Connection: Upgrade
                   # Host: example.com
                   # Origin: http://example.com
                   # Sec-WebSocket-Key1: 18x 6]8vM;54 *(5:  {   U1]8  z [  8
                   # Sec-WebSocket-Key2: 1_ tx7X d  <  nw  334J702) 7]o}` 0
                   #
                   # Tm[K T2u

    # Parse server response
    $h->parse(<<"EOF");
    HTTP/1.1 101 WebSocket Protocol Handshake
    Upgrade: WebSocket
    Connection: Upgrade
    Sec-WebSocket-Origin: http://example.com
    Sec-WebSocket-Location: ws://example.com/demo

    fQJ,fN/4F4!~K~MH
    EOF

    $h->error;   # Check if there were any errors
    $h->is_done; # Returns 1

## Parsing and constructing WebSocket frames

Frame class is implemented as an iterator that is feeded with data chunks and
returns a message by message on the `next` call.

    # Create frame
    my $frame = Protocol::WebSocket::Frame->new('123');
    $frame->to_string; # \x00123\xff

    # Parse frames
    my $frame = Protocol::WebSocket::Frame->new;
    $frame->append("123\x00foo\xff56\x00bar\xff789");
    $f->next; # foo
    $f->next; # bar

## Source code

Source code is available via GitHub [http://github.com/vti/protocol-websocket](http://github.com/vti/protocol-websocket).

## Examples

In the next article I am going to show some real life examples on how to use
this module with various event loops and servers. Stay tuned!
