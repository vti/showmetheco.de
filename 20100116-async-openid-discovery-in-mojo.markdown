Title: Async OpenID discovery in Mojo
Tags: Perl, Mojo, OpenID
Comments: no

While still working on my async OpenID implementation Protocol::OpenID that can
be found at [http://github.com/vti/protocol-openid](http://github.com/vti/protocol-openid) I was refactoring discovery
part and tried to test it on live examples. Thanks to [Mojo::Client](https://metacpan.org/pod/Mojo::Client), that now
has async interface, the code looks very clean.

[cut] See the code

    #!/usr/bin/perl

    use strict;
    use warnings;

    use Protocol::OpenID::Identifier;
    use Protocol::OpenID::Discoverer;

    use Mojo::Client;
    use Mojo::Transaction::Single;

    my $client = Mojo::Client->new(max_redirects => 3);

    my $discoverer = Protocol::OpenID::Discoverer->new(
        http_req_cb => sub {
            my ($url, $method, $headers, $body, $cb) = @_;

            $client->get(
                $url => $headers => sub {
                    my ($self, $tx) = @_;

                    my $res = $tx->res;

                    $cb->(
                        $tx->req->url->to_string, $res->code,
                        $res->headers->to_hash,   $res->body
                    );
                }
            )->process;
        }
    );

    my $identifier = Protocol::OpenID::Identifier->new(shift @ARGV);

    $discoverer->discover(
        $identifier => sub {
            my ($self, $discovery) = @_;

            if ($discovery) {
                use Data::Dumper;
                warn Dumper $discovery;
            }
            else {
                warn 'Error: ' . $self->error;
            }
        }
    );

Important thing to remember is that `$client` should be created outside of closures.
