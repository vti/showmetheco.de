Title: Mojolicious, Async::ORM and DBSlayer
Tags: Perl, Mojolicious, async, orm, dbslayer
Comments: no

After experimenting with Redis in [Mojo](https://metacpan.org/pod/Mojo) ([http://github.com/vti/mojox-socket\_stream](http://github.com/vti/mojox-socket_stream)) I wanted to do something asynchronously with [DBI](https://metacpan.org/pod/DBI) also, using my another [Async::ORM](https://metacpan.org/pod/Async::ORM) module. But apparently doing async with [DBI](https://metacpan.org/pod/DBI) is not that easy. By itself [DBI](https://metacpan.org/pod/DBI) blocks everything and thus there are a few workarounds on CPAN with forks, sockets and pipes like [AnyEvent::DBI](https://metacpan.org/pod/AnyEvent::DBI), [POE::Component::EasyDBI](https://metacpan.org/pod/POE::Component::EasyDBI), [POE::Component::SimpleDBI](https://metacpan.org/pod/POE::Component::SimpleDBI), [IO::Lambda::DBI](https://metacpan.org/pod/IO::Lambda::DBI) and others. That is why I have [AnyEvent::DBI](https://metacpan.org/pod/AnyEvent::DBI) driver inside of [Async::ORM](https://metacpan.org/pod/Async::ORM). But I wanted to use [Mojo::IOLoop](https://metacpan.org/pod/Mojo::IOLoop) as an event loop.

Writing another forking hack module is cool but I am not that smart, that's why I googled with a hope to find some event loop independent async [DBI](https://metacpan.org/pod/DBI). And I found dbslayer ([http://code.nytimes.com/projects/dbslayer/wiki](http://code.nytimes.com/projects/dbslayer/wiki)). DBSlayer is a proxy between HTTP+JSON and MySQL. You send your SQL query as a GET request in JSON format and get JSON response. So what I needed is a http client that can send json requests and parse json responses. [Mojo](https://metacpan.org/pod/Mojo) has got both.

[cut] See more code

So I've written DBSlayer driver for [Async::ORM](https://metacpan.org/pod/Async::ORM) that talks to DBSlayer server. Implementation details are not that interesting, but interesting part is in using [Mojo::Client](https://metacpan.org/pod/Mojo::Client) and [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON).

Start dbslayer server

    dbslayer -c myconf.cnf -s localhost -u foo -x bar

Create an [Async::ORM::DBI::DBSlayer](https://metacpan.org/pod/Async::ORM::DBI::DBSlayer) handler

    my $client = Mojo::Client->new;
    my $json   = Mojo::JSON->new;

    my $dbh = Async::ORM::DBI::DBSlayer->new(
        database    => 'async_orm',
        json_encode => sub { $json->encode(@_) },
        json_decode => sub { $json->decode(@_) },
        http_req_cb => sub {
            my ($url, $method, $headers, $body, $cb) = @_;

            $url = Mojo::URL->new($url);
            $url->query($body);

            $client->get(
                $url->to_string => sub {
                    my ($self, $tx) = @_;

                    $cb->(
                        $url, $tx->res->code, $tx->res->headers->to_hash,
                        $tx->res->body
                    );
                }
            )->process;
        }
    );

`http_req_cb` is a callback for making requests to dbslayer server, `json_*` utilities are for parsing JSON.

Then use it in async [Async::ORM](https://metacpan.org/pod/Async::ORM) manner.

    Article->new(title => 'foo')->create(
        $dbh => sub {
            my ($dbh, $article_) = @_;

        }
    );

This way I can use [Mojo::IOLoop](https://metacpan.org/pod/Mojo::IOLoop) (which means in every [Mojolicious](https://metacpan.org/pod/Mojolicious) application) for managing [DBI](https://metacpan.org/pod/DBI) queries inside of [Async::ORM](https://metacpan.org/pod/Async::ORM)!

You can find the latest code at github [http://github.com/vti/async-orm/](http://github.com/vti/async-orm/).
