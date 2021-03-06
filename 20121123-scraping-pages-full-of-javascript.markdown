Title: Scraping pages full of JavaScript
Tags: Perl, scraping, javascript

When scraping internet websites I use [Web::Scraper](https://metacpan.org/pod/Web::Scraper). It's simple, readable and
very handy. I can use XPath and CSS3 selectors or can get raw HTML and parse it
with regexps when in despair. But sometimes the pages are full of JavaScript
that is required to be run in order to modify the DOM, do some adjustments and
so on. Good thing [Web::Scraper](https://metacpan.org/pod/Web::Scraper) does not only accepts URLs but the raw content
also. So let's use WebKit to render the HTML!

[cut]

As all the good things WebKit Perl bindings are already on CPAN. I used
[Gtk3::WebKit](https://metacpan.org/pod/Gtk3::WebKit) and it worked very well. Here is the fetcher class:

    package Fetcher::WebKit;

    use strict;
    use warnings;

    use Capture::Tiny qw(capture);

    use Gtk3 -init;
    use Gtk3::WebKit ':xpath_results';

    sub new {
        my $class = shift;

        my $self = {@_};
        bless $self, $class;

        return $self;
    }

    sub fetch {
        my $self = shift;
        my ($url) = @_;

        my $loop = Glib::MainLoop->new();

        my $view = Gtk3::WebKit::WebView->new();

        $self->_register_requests;

        $view->signal_connect(
            'notify::load-status' => \&_load_status_cb,
            [$self, $loop]
        );
        $view->load_uri($url);

        # Shut the #$%# up!
        capture { $loop->run() };

        my $doc = $view->get_dom_document;
        return $doc->get_body->get_outer_html;
    }

    sub _register_requests {
        my $self = shift;

        my $session = Gtk3::WebKit->get_default_session();
        $self->{requests_started}  = 0;
        $self->{requests_finished} = 0;
        $session->signal_connect(
            'request-started' => sub {
                my ($session, $message, $socket) = @_;

                ++$self->{requests_started};
                $message->signal_connect(
                    "finished" => sub {
                        my ($message) = @_;

                        ++$self->{requests_finished};
                    }
                );
            }
        );
    }

    sub _load_status_cb {
        my ($self, $loop, $resources) = @{pop @_};
        my ($view) = @_;

        my $uri = $view->get_uri or return;
        return unless $view->get_load_status eq 'finished';

        my $frame       = $view->get_main_frame;
        my $data_source = $frame->get_data_source;
        return if $data_source->is_loading;

        Glib::Idle->add(
            sub {
                return 1
                  unless $self->{requests_started}
                  and $self->{requests_started} == $self->{requests_finished};

                eval { $loop->quit(); };
            }
        );
    }

    1;

And I can do:

    my $fetcher = Fetcher::WebKit->new;
    my $content = $fetcher->fetch('http://some.url');

    my $scraper = scraper {
        process 'div', 'info' => scraper {
            process 'h1',         title => 'TEXT';
            process 'span.date', date  => [
                'TEXT',
                sub { join '-', reverse split /\./ }
            ];

            process 'p', 'content[]' => 'TEXT';
        };
    };

    my $info = $scraper->scrape($content)->{info};

And no JavaScript can stop you!
