Title: Mojolicious and WebSockets
Tags: Perl, Mojolicious, WebSockets
Comments: no

So here is my first attempt to try this new awesome WebSocket technology. And since [Mojolicious](https://metacpan.org/pod/Mojolicious) is already compatible I couldn't wait too long.

[cut]

David Davis has very good examples ([http://github.com/xantus/mojo-websocket-examples](http://github.com/xantus/mojo-websocket-examples)) and they helped me to start very quickly. Thanks to him we already have flash policy server written in mojolicious. I just copied that into my repository. All the Flash files and scripts can be found in gimit's repository ([http://github.com/gimite/web-socket-js](http://github.com/gimite/web-socket-js)). Flash policy server and friends are needed for non websocket native browsers.
Otherwise they are not used.

All I had to do was to start flash policy server,

    sudo ./script/flash-policy-server

to include in my template flash workaround,

    <script type="text/javascript">
        // Only load the flash fallback when needed
        if (!('WebSocket' in window)) {
            document.write([
                '<scr'+'ipt type="text/javascript" src="web-socket-js/swfobject.js"></scr'+'ipt>',
                '<scr'+'ipt type="text/javascript" src="web-socket-js/FABridge.js"></scr'+'ipt>',
                '<scr'+'ipt type="text/javascript" src="web-socket-js/web_socket.js"></scr'+'ipt>'
            ].join(''));
        }
    </script>

and I was ready to go.

I've decided to write a slideshow presentation software. I did it because I thought it would be fun to have a slideshow server that could be used by other people to see the slides when there is no projector or to watch it from the internet, chat and have fun.

The code can be found in my github repository

    http://github.com/vti/showmetheslides

The issues that I've faced are really minor but really important for me.

The first is a necessity of encoding JavaScript strings into UTF-8 before sending over WebSocket. This can be done via encoding function:

    function toUTF(string) {
        string = string.replace(/\r\n/g, "\n");
        var utftext = "";

        for (var n = 0; n < string.length; n++) {
            var c = string.charCodeAt(n);

            if (c < 128) {
                utftext += String.fromCharCode(c);
            }
            else if((c > 127) && (c < 2048)) {
                utftext += String.fromCharCode((c >> 6) | 192);
                utftext += String.fromCharCode((c & 63) | 128);
            }
            else {
                utftext += String.fromCharCode((c >> 12) | 224);
                utftext += String.fromCharCode(((c >> 6) & 63) | 128);
                utftext += String.fromCharCode((c & 63) | 128);
            }
        }

        return utftext;
    }

And before sending to WebSocket:

    ws.send(toUTF(string_with_utf));

Another issue was organizing broadcast messages to all connected clients. This was done by using a queue of connections. All I had to do is just to save WebSocket transaction and then use it to send WebSocket messages.

I used Perl hashes. The keys are stringified controller objects:

    websocket '/' => sub {
        my $self = shift;
        my $tx = $self->tx;

        my $cid = "$tx";

        $clients->{$cid} = $tx;

        $self->receive_message(
            sub {
                my ($self, $message) = @_;
            }
        );
    };

And then broadcast looked like:

    foreach my $cid (keys %$clients) {
        $clients->{$cid}->send_json($data);
    }

Other important thing is not to forget to delete transaction from the queue on WebSocket disconnect:

    $self->finished(
        sub {
            delete $clients->{$cid};

            app->log->debug('Client disconnected');
        }
    );

To sum all the work I must say that [Mojolicious](https://metacpan.org/pod/Mojolicious) WebSocket implementation works flawless and I actually spent more time programming JS and writing CSS styles :)

Try WebSockets, it's easy!
