Title: Pastelicious
Tags: perl, mojo
Comments: no

Yet another pastebin project, but implemented on top of [Mojolicious::Lite](https://metacpan.org/pod/Mojolicious::Lite). The codebase is actually very simple but shows the best [Mojolicious](https://metacpan.org/pod/Mojolicious) features.

[cut]

## File-based model

All pastes are saved on the hard disk, thus no database is required. Each paste has unique SHA-1 hex name. Pastes can be private and thus only you know the url and only you can delete them. This way no password protection is required.

## JSON configuration

[Mojo::JSON](https://metacpan.org/pod/Mojo::JSON) parser makes app configuration trivial to implement. In Pastelicious loading plugins and including Perl libraries paths (soon languages and syntax highlighting) is done via configuration file.

    {
        "perl5lib" : "/home/vti/local/lib/perl5",
        "plugins_namespaces" : ["Bootylicious::Plugin"],
        "plugins" : [ 
            "google_analytics", {"urchin" : "UA-1234567-8"}
        ]   
    }

## Static dispatcher

[MojoX::Dispatcher::Static](https://metacpan.org/pod/MojoX::Dispatcher::Static) can be used not only while developing but also when you want to serve a static file from your application. This way in Pastelicious I serve the source code.

    $self->render_static('../' . basename($0));

## Routes

Routes is a readable and customizable way to match application urls to controllers. Stash is automatically populated by captures and everything else is just trivial.

    get '/source' => sub {
        my $self = shift;

        $self->stash(rendered => 1); 
        $self->app->static->serve($self, '../' . basename($0));
    } => 'source';

## Formats

Having format support in [MojoX::Renderer](https://metacpan.org/pod/MojoX::Renderer) it is easy to have different formats in which you give a user the data. Based on url it is possible to render an appropriate template.

    http://showmetheco.de/0761ddc714409b83d75afee83190f2b1af2a25a7
    # Render view.html.ep

    http://showmetheco.de/0761ddc714409b83d75afee83190f2b1af2a25a7.raw
    # Render view.raw.ep

## Templates

Perlish [Mojo::Template](https://metacpan.org/pod/Mojo::Template) templates are powerful and don't require learning anything new. In Pastelicious I've used new `ep` templates that have stash variables auto initializition, helpers etc.

    % if (my $author = $metadata->{author}) {
    <tr>
    <td class="label">Author:</td><td><%= $author %></td>
    </tr>
    % }  

## Renderer helpers

[MojoX::Renderer](https://metacpan.org/pod/MojoX::Renderer) helpers introduce Perl callback subroutines that can be used in templates. In Pastelicious I use them to get last pastes and available languages.

    % foreach my $id (get_last_pastes) {
      <a href="<%= url_for view => id => $id %>"><%= $id %></a><br />
    % }

## Plugins

[Mojolicious::Plugins](https://metacpan.org/pod/Mojolicious::Plugins) hook model being very customizable gives you ability to use third party modules without changing application code. For example, at [http://showmetheco.de](http://showmetheco.de) I use [Bootylicous::Plugin::GoogleAnalytics](https://metacpan.org/pod/Bootylicous::Plugin::GoogleAnalytics) for collecting statistics.

## Deployment

Deployment of [Mojo](https://metacpan.org/pod/Mojo) application is not hard, since [Mojo](https://metacpan.org/pod/Mojo) has no dependencies and supports CGI, FastCGI, mod\_perl, embedded daemon and daemon\_prefork servers.

## Source code and Demonstration

You can find Pastelicious source code at [http://github.com/vti/pastelicious](http://github.com/vti/pastelicious) and see it in action at [http://showmetheco.de](http://showmetheco.de).
