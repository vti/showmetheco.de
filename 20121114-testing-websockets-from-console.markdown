Title: Testing WebSockets from console
Tags: perl, websockets, console

Since August 2012 [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) is shipped with a websocket console
application (under `util/` directory), that can be used for testing, debugging
and/or learning more about websockets.

[cut]

## Connecting to server

    $ perl wsconsole ws://127.0.0.1:5000
    ! Using version 'draft-ietf-hybi-17'
    ! Connecting to '127.0.0.1:5000'
    ! Connected
    > Writing handshake
    > GET / HTTP/1.1
    > Upgrade: WebSocket
    > Connection: Upgrade
    > Host: 127.0.0.1:5000
    > Origin: http://127.0.0.1:5000
    > Sec-WebSocket-Key: +5kmx+0mCkQhYSN6BOCZuA==
    > Sec-WebSocket-Version: 13
    >
    >
    > < Reading handshake
    > HTTP/1.1 101 WebSocket Protocol Handshake
    > Upgrade: WebSocket
    > Connection: Upgrade
    > Sec-WebSocket-Accept: +/AulSjX1AUSVySlyEk+Rj/uvMQ=
    >

As you can see there is some debug info about the connection, its version and so
on. You can see the handshake also.

When connecting you can specify the WebSocket version. This way it is possible
to check if the server supports all of them.

    $ perl wsconsole ws://127.0.0.1:5000 draft-ietf-hybi-10
    ! Using version 'draft-ietf-hybi-10'
    ! Connecting to '127.0.0.1:5000'
    ! Connected
    > Writing handshake
    > GET / HTTP/1.1
    > Upgrade: WebSocket
    > Connection: Upgrade
    > Host: 127.0.0.1:5000
    > Sec-WebSocket-Origin: http://127.0.0.1:5000
    > Sec-WebSocket-Key: 82d+qtJEEDheMJPOecIWeQ==
    > Sec-WebSocket-Version: 8
    >
    >
    > < Reading handshake
    > HTTP/1.1 101 WebSocket Protocol Handshake
    > Upgrade: WebSocket
    > Connection: Upgrade
    > Sec-WebSocket-Accept: 4hUMi7VdcFzz8mupKMuoi5uBits=
    >

As you can see the handshake is different.

## Sending frames

As you type the application doesn't send anything, but does so as soon as you
hit CTRL-D, which is 'end of file'.

    hello> Writing frame
    vvvvvvvvvv
    [0000]   81 85 A8 78  2F A3 C0 1D  43 CF C7                   ...x /... C..

    ^^^^^^^^^^

The frame is dumped as hex (thanks to [Devel::Hexdump](https://metacpan.org/pod/Devel::Hexdump)).

## Receiving frames

Incoming frames are printed as soon as they are received.

    < Reading frame
    vvvvvvvvvv
    [0000]   81 05 68 65  6C 6C 6F                                ..he llo

    ^^^^^^^^^^

This is not very convinient when you type of course, maybe this will be improved
some day...
