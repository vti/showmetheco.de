Title: Async OpenID RelyingParty in Mojo
Tags: Perl, Mojo, OpenID
Comments: no

I keep working on async framework agnostic Protocol::OpenID ([http://github.com/vti/protocol-openid](http://github.com/vti/protocol-openid)) Perl module, and just finished OpenID associations. It is always nice to have some real life tests, and with [Mojo](https://metacpan.org/pod/Mojo) it is fun and easy.

[cut] See RelyingParty implementation in Mojo

Not only great [Mojo](https://metacpan.org/pod/Mojo) async server is used here, but also [Mojo::Client](https://metacpan.org/pod/Mojo::Client) which has embedded redirection support. Testing is so easy, no need for standalone server setup, just Perl and [Mojo](https://metacpan.org/pod/Mojo)!.

To run this code you will need [Crypt::DH](https://metacpan.org/pod/Crypt::DH), [Protocol::Yadis](https://metacpan.org/pod/Protocol::Yadis), [Mojo](https://metacpan.org/pod/Mojo) and Protocol::OpenID with is not yet released, but can be found at [http://github.com/vti/protocol-openid](http://github.com/vti/protocol-openid).

    #!/usr/bin/perl

    use Mojolicious::Lite;
    use Mojo::Client;
    use Mojo::Transaction::Single;
    use Mojo::Parameters;

    use Protocol::OpenID::RP;

    our $STORE = {};

    app->client->max_redirects(3);
    my $rp = Protocol::OpenID::RP->new(

        # Callback for downloading pages
        http_req_cb => sub {
            my ($url, $method, $headers, $body, $cb) = @_;

            my $tx = Mojo::Transaction::Single->new;
            $tx->req->method($method || 'GET');
            $tx->req->url->parse($url);

            $tx->req->headers->from_hash($headers);

            if ($tx->req->method eq 'POST') {

                # Can't use multipart, specs say it should be x-www-encoded POST
                $tx->req->headers->content_type(
                    'application/x-www-form-urlencoded');

                $tx->req->body(Mojo::Parameters->new(%$body));
            }

            app->client->process(
                $tx => sub {
                    my ($client, $tx) = @_;

                    $cb->(
                        $tx->req->url->to_string, $tx->res->code,
                        $tx->res->headers->to_hash,
                        $tx->res->body
                    );
                }
            );
        },

        # Simple memory storage
        store_cb => sub {
            my ($key, $data, $cb) = @_;

            $STORE->{$key} = $data;

            $cb->();
        },
        find_cb => sub {
            my ($key, $cb) = @_;

            $cb->($STORE->{$key});
        },
        remove_cb => sub {
            my ($key, $cb) = @_;

            delete $STORE->{$key};

            $cb->();
        }
    );

    any [qw/get post/] =>'/' => {errors => {}} => sub {
        my $self = shift;

        my $openid_identifier = $self->param('openid_identifier');

        if (defined $openid_identifier) {
            if (!$openid_identifier) {
                $self->stash(errors => {openid_identifier => 'Required'});
                return;
            }
        }

        $rp->return_to($self->url_for->to_abs);

        # Pause transaction, so we can download stuff from the internet
        $self->pause;

        $rp->authenticate(
            $self->req->params->to_hash,
            sub {
                my ($rp, $tx) = @_;

                # Resume transaction
                $self->resume;

                if ($tx->error) {
                    $self->stash->{errors}->{openid_identifier} = $tx->error;
                }
                elsif ($tx->state eq 'redirect') {
                    my $location = $tx->op_endpoint;
                    $location = Mojo::URL->new($location);
                    $location->query(%{$tx->request->to_hash});

                    $self->redirect_to("$location");
                    return;
                }

                elsif ($tx->state eq 'cancel') {
                    $self->stash->{errors}->{openid_identifier} = 'cancel';
                }

                elsif ($tx->state eq 'setup_needed') {
                    $self->res->body('setup_needed');
                }

                elsif ($tx->state eq 'success') {
                    $self->res->body('verified');
                }

                $self->render;
            }
        );
    } => 'index';

    shagadelic('daemon_prefork');

    __DATA__

    @@ index.html.ep
    <div style="margin:0px auto;">
    <form method="post">
    OpenID: <input name="openid_identifier" /><br />
    % if (my $od_e = $errors->{openid_identifier}) {
    <div style="color:red;"><%= $od_e %></div>
    % }
    <input type="submit" value="Login" />
    </form>
    </div>
