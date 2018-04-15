Title: Building a multilingual website with Mojolicious
Tags: Perl, Mojolicious, I18N
Comments: no

For a long time there was the [MojoX::Locale::Maketext](https://metacpan.org/pod/MojoX::Locale::Maketext) module. But it is currently deprecated since [Mojolicious](https://metacpan.org/pod/Mojolicious) has [Mojolicious::Plugin::I18n](https://metacpan.org/pod/Mojolicious::Plugin::I18n) in its core. Today we will try to use it to create a multilingual website.

[cut]

At first we setup charset plugin so unicode data is correctly displayed:

    $self->plugin(charset => {charset => 'utf8'});

Then we create lexicon files for every language we want to have. Let `en` be the default one.

    package MyApp::I18N::en;

    use base 'MyApp::I18N';

    our %Lexicon = ( _AUTO => 1); 

    1;

    package MyApp::I18N::ru;

    use base 'MyApp::I18N';

    # Don't forget if we have unicode symbols in package file
    use utf8;

    our %Lexicon = ( 
        'Add'    => 'Добавить',
        'Remove' => 'Удалить'
    );

    1;

Then setup i18n plugin:

    $self->plugin(i18n => {namespace => 'MyApp::I18N'});

In templates now we have `l` helper:

    <%=l 'Add' %>

[Mojolicious](https://metacpan.org/pod/Mojolicious) will guess what language user wants looking into `Accept` headers.

That could be it. But it would be better if we had prefixed urls for every language.

    /en/welcome
    /ru/welcome

Without using a bridge we set up a `after_static_dispatch` hook that will cut the language tag and set it as the current language.

       $self->plugins->add_hook(after_static_dispatch => sub {
            my ($self, $c) = @_; 

            # We don't want to parse static files urls
            return if $c->res->code;

            if (my $path = $c->tx->req->url->path) {
                my $part = $path->parts->[0];

                if ($part && grep { $part eq $_ } @{$config->{languages}}) {
                    shift @{$path->parts};

                    $c->app->log->debug("Found language $part in url");

                    $c->stash->{i18n}->languages($part);
                }
            }
        }
    ); 

Also we want `url_for` to build the correct urls on behalf of the current language. In our [Mojolicious](https://metacpan.org/pod/Mojolicious) controller we overwrite `url_for` method:

    sub url_for {
        my $self = shift;

        my $url = $self->SUPER::url_for(@_);

        my $language = $self->stash->{i18n}->languages;

        unshift @{$url->path->parts}, $language;

        return $url;
    }

Now we have automatic language detection and language urls that will point to the right translation if you want to share a link in a specific language.

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 29:

    Non-ASCII character seen before =encoding in ''Добавить','. Assuming UTF-8
