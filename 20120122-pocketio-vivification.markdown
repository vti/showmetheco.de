Title: PocketIO vivification
Tags: Perl, pocketio, socketio

Since `0.10` [PocketIO](https://metacpan.org/pod/PocketIO) doesn't require [Plack](https://metacpan.org/pod/Plack) and has a new dependency
[Protocol::SocketIO](https://metacpan.org/pod/Protocol::SocketIO).

[cut]

I've extracted all the SocketIO implementation functionality like path parsing,
handshake and message building into a separate distribution
[Protocol::SocketIO](https://metacpan.org/pod/Protocol::SocketIO). This way it can be uses outside of [PocketIO](https://metacpan.org/pod/PocketIO) and
[AnyEvent](https://metacpan.org/pod/AnyEvent).

[PocketIO](https://metacpan.org/pod/PocketIO) now doesn't require [Plack](https://metacpan.org/pod/Plack), it requires only a [PSGI](https://metacpan.org/pod/PSGI) compatible
server with `psgix.io` support (like [Feersum](https://metacpan.org/pod/Feersum)). This is much easier to deploy.

Why? I've got an email on why [PocketIO](https://metacpan.org/pod/PocketIO) is so highly coupled with [Plack](https://metacpan.org/pod/Plack) and
moreover why can't SocketIO protocol implementation can't be used outside of
[PocketIO](https://metacpan.org/pod/PocketIO)?

And I thought the same way. Many frameworks for various functionality I've used
were forcing their way of doing things, and thus replacing flexibility with
vendor lock-in. Particular implementation shoudn't stop you from replacing it
with another one later on. And some libraries (speaking not about [Plack](https://metacpan.org/pod/Plack) here
:) go too far by introducing their own base classes that you have to inherit
from, exception mechanizms that behave not in the common way and lots of code
that you'll never need.

At work when using a CPAN module we at least write an adaptor with interface and
functionality we need, which saves us from the replacement pain or unexpected
release lacking backwards compatibility.

More KISS and YAGNI for everyone.
