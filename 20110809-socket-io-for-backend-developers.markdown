Title: Socket.IO for backend developers
Tags: Perl
Comments: no

There is an official Socket.IO specification document, but it doesn't tell you
the implementation details of every transport. When implementing a server side
of Socket.IO you have to use `tcpdump` or `wireshark` to capture packets
and actually see what you have to send and receive. So here are the details that
I collected when implementing [http://github.com/vti/pocketio](http://github.com/vti/pocketio).

[cut]

First, read the specification [https://github.com/learnboost/socket.io-spec](https://github.com/learnboost/socket.io-spec).

## Handshake

In order to make some browser debug tools like Firebug happy specify a correct
`Content-Type` for the handshake response. I set `text/plain`.

    HTTP/1.1 200 OK
    Content-Type: text/plain
    Connection: keep-alive
    Content-Length: 73

    1234567890:15:25:websocket,flashsocket,htmlfile,xhr-polling,jsonp-polling

### Tranport priority

The recommented transport priority is:

    websocket,flashsocket,htmlfile,xhr-polling,jsonp-polling

### Cross-Domain connections

When you want to run your Socket.IO server on a different port or domain from
the main website, you have to make sure it supports cross-domain jsonp calls.

So on this handshake request:

    GET /socket.io/1/?t=1312877027739&jsonp=0 HTTP/1.1
    Host: localhost:5000
    Connection: keep-alive

you should response with:

    HTTP/1.1 200 OK
    Content-Type: application/javascript
    Content-Length: 90

    io.j[0]("1234567890:15:25:websocket,flashsocket,htmlfile,xhr-polling,jsonp-polling");

`0` is from request `jsonp=0` query string.

Every next response should contain the following header:

    Access-Control-Allow-Origin: *

## Flash policy server

When using a Flash fallback you must run a Flash policy server
([https://github.com/keroyonn/p5-AnyEvent-FlashSocketPolicy](https://github.com/keroyonn/p5-AnyEvent-FlashSocketPolicy) or
[https://github.com/vti/pocketio/blob/master/examples/flash-policy-server](https://github.com/vti/pocketio/blob/master/examples/flash-policy-server)).
This can be a standalone server that listens on `843` port (must be run as
root) or an inline server that answers not only `HTTP` but also Flash requests
([Fliggy](https://metacpan.org/pod/Fliggy)).

## Transports

Now you know how to do a handshake, how to parse and build messages. As you
already know we have to implement the following transports:

    WebSocket
    Flashsocket
    XHR Polling
    XHR Multipart
    Htmlfile
    JSONP polling

`WebSocket` and `Flashsocket` are easy, just use [Protocol::WebSocket](https://metacpan.org/pod/Protocol::WebSocket) for
parsing `WebSocket` handshake and frames. These two transports are exactly the
same on the server side.

`XHR Multipart` is disabled in the latest Socket.IO version. I haven't really
dug into this problem, but I hope it will be reenabled in the next versions
(AFAIK it affects only Firefox 3.x, but you can still use `XHR Polling` there).

So we are left with `XHR Polling`, `JSONP Polling` and `Htmlfile`. These
transports are unidirectional. This means we have to use one channel for sending
messages and one channel for receiving them. Moreover the server channel is a
`GET` request with a delayed response (`XHR Polling` and `JSONP Polling`) or
streaming response (`Htmlfile`) and the client channel is a sequence of `POST`
requests with simple confirmation responses.

`XHR Polling` and `JSONP Polling` transports reconnect after the message from
the server to the client was sent. So you have to cache the messages that are
sent during reconnection and then send them sequentially when the connection is
established.

A detailed explanation of every transport follows.

### XHR Polling

#### Server -> Client

When `XHR Polling` connection is established a `GET` request is made awaiting
for a delayed response from the server.

    GET /socket.io/1/xhr-polling/15725592722050266739/?t=1312200210467 HTTP/1.1
    Host: 127.0.0.1:5000
    Connection: Keep-Alive

When the server wants to send a message it responses with something like:

    HTTP/1.1 200 OK
    Content-Type: text/plain
    Content-Length: 47

    5:::{"args":[{"vti":"vti"}],"name":"nicknames"}

and closes the connection. It could be a chunked response by the way.

#### Client -> Server

When a client wants to send a message it creates a `POST` request:

    POST /socket.io/1/xhr-polling/15725592722050266739?t1312200250539 HTTP/1.1
    Host: 127.0.0.1:5000
    Content-Length: 45
    Connection: Keep-Alive

    5:::{"name":"user message","args":["hello!"]}

And should receive a simple `200` response from the server:

    HTTP/1.0 200 OK
    Content-Length: 1

    1

### JSONP Polling

`JSONP Polling` is similar to `XHR Polling`. You just have to wrap messages
into `JavaScript`.

#### Server -> Client

The server channel is created as a `GET` request awaiting for a delayed response:

    GET /socket.io/1/jsonp-polling/4168417008084729/?t=1312874409449&i=0 HTTP/1.1
    Host: localhost:5000
    Connection: keep-alive

When the server wants to send a message it responses with (notice the correct
`Content-Type`):

    HTTP/1.1 200 OK^M
    Content-Type: text/javascript; charset=UTF-8
    Content-Length: 69

    io.j[0]("5:::{\"args\":[{\"vti\":\"vti\"}],\"name\":\"nicknames\"}");

And closes the connection. Noticed `io.j[0]("a message goes here")`? Don't forget
to escape quotes when wrapping your message.

#### Client -> Server

When a client wants to send a message it makes a `POST` request:

    POST /socket.io/1/jsonp-polling/4168417008084729?t=1312874410509&i=0 HTTP/1.1
    Host: localhost:5000
    Connection: keep-alive
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 85

    d=5%3A%3A%3A%7B%22name%22%3A%22user+message%22%2C%22args%22%3A%5B%22hello%21%22%5D%7D

As you can see the request now is `x-www-form-urlencoded` with a `d` parameter
with encoded message. Don't forget to decode the message on the server.

And the client should receive a simple `200` response from the server:

    HTTP/1.0 200 OK
    Content-Length: 1

    1

### Htmlfile

`Htmlfile` is a tranport for Internet Explorer (read "some nasty hacks
follow").

#### Server -> Client

The server channel is created as a `GET` request and a streaming response
(server does not close the connection when sending a message to the client, it
just pushes it):

    GET /socket.io/1/htmlfile/4168417008084729/?t=1312874409449 HTTP/1.1
    Host: localhost:5000
    Connection: keep-alive

The server responses with a chunked streaming response:

    HTTP/1.1 200 OK
    Content-Type: text/html
    Connection: keep-alive
    Transfer-Encoding: chunked

    100
    <html><body><script>var _ = function (msg) { parent.s._(msg, document); };</script>

Followed by `173` spaces (I am sure you can google why) and an empty line.

When the server wants to send a message it pushes a chunk to the stream:

    50
    <script>_("5:::{\"args\":[{\"vti\":\"vti\"}],\"name\":\"nicknames\"}");</script>

You have to wrap your message into `script` tags and `_("your messages")`
function. Don't forget to escape quotes.

#### Client -> Server

When a client wants to send a message it makes a `POST` request:

    POST /socket.io/1/xhr-polling/15725592722050266739?t1312200250539 HTTP/1.1
    Host: 127.0.0.1:5000
    Content-Length: 45
    Connection: Keep-Alive

    5:::{"name":"user message","args":["hello!"]}

And the client should receive a simple `200` response from the server:

    HTTP/1.0 200 OK
    Content-Length: 1

    1

## Horizontal scaling

When using Nginx with `tcp_proxy_module`
([https://github.com/yaoweibin/nginx\_tcp\_proxy\_module](https://github.com/yaoweibin/nginx_tcp_proxy_module)) for proxying Socket.IO
connections (how to setup read more at
[http://www.letseehere.com/reverse-proxy-web-sockets](http://www.letseehere.com/reverse-proxy-web-sockets)) you can't be sure that
the next request will come to the same process and thus you have to make sure
different processes can communicate, exchanging connections, sending broadcast
mesages etc (more at
[http://stackoverflow.com/questions/5944714/how-can-i-scale-socket-io](http://stackoverflow.com/questions/5944714/how-can-i-scale-socket-io)).

The guys who made Socket.IO have a cluster solution
[http://learnboost.github.com/cluster/](http://learnboost.github.com/cluster/) for NodeJS. Maybe this can be brought
into Perl too.

Scaling support for PocketIO is on the roadmap.

## Comments

If you found any error or something is not detailed enough, fill free to
comment below.
