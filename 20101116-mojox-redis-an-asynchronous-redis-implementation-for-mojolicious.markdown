Title: MojoX::Redis: an asynchronous Redis implementation for Mojolicious
Author: und3f
Tags: Perl, Mojolicious, Redis
Comments: no

Author of this post is und3f, author of [MojoX::Redis](https://metacpan.org/pod/MojoX::Redis). You can follow him on
Twitter [http://twitter.com/und3f](http://twitter.com/und3f) and/or GitHub [http://github.com/und3f](http://github.com/und3f).

Today's topic is [MojoX::Redis](https://metacpan.org/pod/MojoX::Redis) module for [Mojolicious](https://metacpan.org/pod/Mojolicious) and asynchronous
commucation with Redis.

[cut]

Essentially Redis is a lightweight but advanced key-value store with great
possibilities. It is available from [http://code.google.com/p/redis/](http://code.google.com/p/redis/). Redis is
similar to memcached but the dataset is not volatile, and values can not only
be strings, like in memcached, but also lists, sets and ordered sets.  All
data types can be manipulated with atomic operations: push/pop elements,
add/remove elements, perform server side union, intersection, difference between
the sets, and so forth. Redis supports different kinds of sorting abilities.

Redis has quite a simple but effective communication protocol, with its
limitations and benefits. It is described in Redis documentation
[http://code.google.com/p/redis/wiki/ProtocolSpecification](http://code.google.com/p/redis/wiki/ProtocolSpecification). Commands within
single connections are executed sequentually - remember about that if you need to
write some results to database and main connection is blocked by long-time
operation like `blpop`.

When to use Redis? Redis is for sure very advanced key-value store, but
you must remember that all operation related to creation and maintenance of
complicated relations are on your own. It is hard to use Redis for
applications with a great amount of relations between data (for example social
networks like FaceBook), but if you need to store unrelated data - Redis is
perfect.

MojoX::Redis is quite simple in usage. First of all we need to create a
MojoX::Redis instance.

    use MojoX::Redis;
    my $redis = MojoX::Redis->new;

Now execute commands, MojoX::Redis will take care about the tcp connection.

    $redis->execute(
        ping => sub {
            my ($redis, $result);

            # In case of error $result will be undefined
            die "Redis error occured: " . $redis->error
                unless defined $result;

            # Profit
            print $result->[0], "\n";
        }
    );

Some Redis commands expect one or more arguments. Also you can ignore callback
if you don't want to ensure that command was executed successfully.

        $redis->execute(
            set => [test => "testok"]
        )->execute(
            get => 'test' => sub { ... }
        );

MojoX::Redis is asynchronous, that's why to start it you have to start
Mojo::IOLoop:

    $redis->ioloop->start;

Here are some examples of using [MojoX::Redis](https://metacpan.org/pod/MojoX::Redis) from a real life application. It
is a daemon that uses Redis for communication. It inputs data from key "hosts"
(which is a List, see Redis data types) and stores results in a key named
"host:$host".

For this example to work you need Redis 2.0 or higher.

    #!/usr/bin/env perl

    use strict;
    use warnings;

    use Mojo::Client;
    use MojoX::Redis;

    my $redis = MojoX::Redis->new;

    # We need new redis connection for results
    my $redis_res = MojoX::Redis->new;

    # ioloop is not running yet, so we need to install singleton loop
    my $client = Mojo::Client->new(ioloop => $redis->ioloop)->async;

    # Just simulate the first command execution to avoid copy-pasting
    &request_host($redis, []);

    # Here the magic starts to work
    $redis->ioloop->start;
    sub request_host {
        my ($redis, $result) = @_;

        # Possibly we got some error from Redis
        die $redis->error unless defined defined $result;

        # Everything is ok - just check the result
        my ($key, $host) = @$result;

        # If host is not defined - command were interrupted by a timeout
        if (defined $host) {
            $client->get(
                $host => sub {
                    my $client = shift;
                    $redis_res->execute(
                        set => ["result:$host" => $client->res->body],
                        sub {
                            print @{$_[1]};
                        }
                    );
                }
            )->start;
        }

        # Execute "blocking pop" command on array "hosts" with timeout of
        # 200 seconds
        $redis->execute(blpop => [hosts => 200] => \&request_host);
    }
