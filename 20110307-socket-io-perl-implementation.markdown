Title: Socket.IO Perl implementation
Tags: perl, socket.io

Socket.IO is a universal way to write realtime web apps that work in every
browser. Socket.IO supports several transports that are chosen during runtime
based on the current browser capabilities:

    * WebSocket
    * Adobe® Flash® Socket
    * AJAX long polling
    * AJAX multipart streaming
    * Forever Iframe
    * JSONP Polling

This way it is possible to write WebSocket-like web apps without bothering much
about the vendor support.

Socket.IO has an official client and various (among official) server
implementations in different languages and frameworks. Here is a new Perl
implementation [http://github.com/vti/plack-middleware-socketio](http://github.com/vti/plack-middleware-socketio) built on top
of [Plack](https://metacpan.org/pod/Plack) as a normal middleware.

Socket.IO++

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 8:

    Non-ASCII character seen before =encoding in 'Adobe®'. Assuming UTF-8
