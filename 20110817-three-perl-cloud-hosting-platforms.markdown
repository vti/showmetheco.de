Title: Three Perl cloud hosting platforms
Tags: Perl, hosting, cloud, PSGI, catalyst, dancer

Recently a lot of cloud hosting platforms started to appear. But most of them
are either bound to a particular language or technology or have quite a limited
Perl support. It is always a joy to see a new hosting company that supports my
favorite language. And now we have even three alternatives!

[cut]

Here is a simple application that we will try to deploy (complete source code
examples are available on github, don't copy-n-paste it ;):

    use strict;
    use warnings;

    use DateTime;

    my $BIRTHDAY_DAY   = 23;
    my $BIRTHDAY_MONTH = 7;

    my $app = sub {
        my $env = shift;

        my $current_time = DateTime->now;

        my $body;
        if (   $current_time->day == $BIRTHDAY_DAY
            && $current_time->month == $BIRTHDAY_MONTH)
        {
            $body = 'Happy birthday!';
        }
        else {
            my $next_birthday = DateTime->new(
                year  => $current_time->year,
                month => $BIRTHDAY_MONTH,
                day   => $BIRTHDAY_DAY
            );

            $next_birthday += DateTime::Duration->new(years => 1)
              if $current_time > $next_birthday;

            my $diff = $next_birthday->subtract_datetime_absolute($current_time);

            $body =
              'Please, wait for just ' . $diff->in_units('seconds') . ' seconds.';
        }

        return [200, ['Content-Length' => length($body)], [$body]];
    };

    $app;

It just checks if today is my birthday and otherwise tells me how many seconds
I should wait :)

## dotCloud

First I tried [http://dotcloud.com](http://dotcloud.com). You have to install a special
Python-driven helper application that allows you to control your services and to
deploy your applications. Actually any [PSGI](https://metacpan.org/pod/PSGI) compliant application can be
pushed.

### Installing CLI

    sudo easy_install pip && sudo pip install dotcloud

### Configuring CLI

    dotcloud

And enter you API key.

### Preparing a project

Clone the project from [https://github.com/vti/dotcloud-perl-example](https://github.com/vti/dotcloud-perl-example).

It should look smth like this:

    dotcloud-perl-example
    ├── dotcloud.yml
    └── mybirthday
        ├── app.psgi
        └── Makefile.PL

`dotcloud.yml`:

    www:
        type: perl
        approot: mybirthday

Make sure you save [PSGI](https://metacpan.org/pod/PSGI) app as `app.psgi`. We have dependencies, so we have
to configure `Makefile.PL`. Official documentation
[http://docs.dotcloud.com/services/perl/](http://docs.dotcloud.com/services/perl/) provides a sample `Makefile.PL` that
looks like:

    PREREQ_PM => {
        'Plack'    => 0.9974,
        'DateTime' => 0
    },

But in reality it should be a complete `Makefile.PL`:

    #!/usr/bin/env perl
    use ExtUtils::MakeMaker;
    WriteMakefile(
        PREREQ_PM => {
            'DateTime'       => 0,
            'Plack::Request' => '0.9976',
        },
    );

### Creating a service

    dotcloud create mybirthday

### Pushing an application

    dotcloud push mybirthday dotcloud-perl-example

    Deployment finished. Your application is available at the following URLs
    www: http://ed3531d3.dotcloud.com/

If you face any problems just run `dotcloud logs mybirthday.www` and you will
get all the logs. It is very easy to spot a problem.

### More examples

- Dancer [http://blogs.perl.org/users/marco\_fontani/2011/04/dancing-on-a-cloud-made-of-pearls.html](http://blogs.perl.org/users/marco_fontani/2011/04/dancing-on-a-cloud-made-of-pearls.html)
- Catalyst [http://onionstand.blogspot.com/2011/04/catalyst-in-cloud.html](http://onionstand.blogspot.com/2011/04/catalyst-in-cloud.html)

## ActiveState Stackato

Then I tried [http://www.activestate.com/cloud](http://www.activestate.com/cloud) platform. This is more like a
personal cloud service. You get a virtual machine image with pre-setup
components. An additional helper application is also available, but you can
actually use a ssh client and run commands on the server. There are some tweaks
with the network setup but they are well described on the documentation page.

### Installing CLI

Download and put somewhere in your `PATH`
`stackato` CLI application [http://community.activestate.com/stackato/download](http://community.activestate.com/stackato/download).

Then download a Virtual Machine image from the same page. I've downloaded
VirtualBox and had no problems.

### Configuring CLI

Just follow the instructions at
[http://community.activestate.com/stackato/documentation/getstarted](http://community.activestate.com/stackato/documentation/getstarted).

### Preparing a project

Clone the project from [https://github.com/vti/activestate-stackato-perl-example](https://github.com/vti/activestate-stackato-perl-example).

Our project should look smth like this:

    activestate-stackato-perl-example
    └── Makefile.PL
    └── app.psgi

Where

- `Makefile.PL` is the same for dotCloud

        #!/usr/bin/env perl
        use ExtUtils::MakeMaker;
        WriteMakefile(
            PREREQ_PM => {
                'DateTime'       => 0,
                'Plack::Request' => '0.9976',
            },
        );

- `app.psgi` is our application

### Pushing an application

    stackato push mybirthday

And then visit your applicaton localy, for example [mybirthday.stackato.local](https://metacpan.org/pod/mybirthday.stackato.local).

### More examples

- Catalyst [https://github.com/ActiveState/stackato-samples/tree/master/perl/catalyst-welcome](https://github.com/ActiveState/stackato-samples/tree/master/perl/catalyst-welcome)

## Fluxflex

Recently I found [http://www.fluxflex.com](http://www.fluxflex.com). Maybe it is not as powerful as
previous solutions, but it doesn't require you to install any additional
programs for pushing your apps, integrates with [http://github.com](http://github.com) and is very
very very cheap. A good choice for simple and medium websites.

### Preparing a project

Clone the project from [https://github.com/vti/fluxflex-perl-example](https://github.com/vti/fluxflex-perl-example).

You can either create a new GitHub repository or just fork the link from above.

    fluxflex-perl-example
    ├── extlib
    └── public_html
        ├── .htaccess
        └── dispatch.fcgi

Where

- `extlib` is a directory for our dependencies.
- `.htaccess`

        Options +FollowSymLinks +ExecCGI
        RewriteEngine On
        RewriteBase /
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ dispatch.fcgi/$1 [QSA,L]
        DirectoryIndex dispatch.fcgi index.html index.htm

- `dispatch.fcgi`

        #!/usr/bin/env perl

        use strict;
        use warnings;

        use Plack::Handler::FCGI;

        my $app = sub {
            # our application
        };

        Plack::Handler::FCGI->new->run($app);

**IMPORTANT**! Do not forget to change `dispatch.fcgi` permissions to `755`.

    chmod +x public_html/dispatch.fcgi

Or you will waste a lot of time trying to figure out what's this all about:

    Premature end of script headers: dispatch.fcgi

### Creating a service

Create a service online and connect to your GitHub project.

### Pushing an application

Just a normal `git push`. Easy. The project will be updated automatically (or
manually via deploy button).

    git push

And then visit you application:

    http://mybirthday.fluxflex.com/

### More examples

- Simple CGI scripts [https://github.com/alg0002/fluxflex\_perl\_cgi\_sample](https://github.com/alg0002/fluxflex_perl_cgi_sample)
- CGI::Application [https://github.com/kyoro/fluxflex\_sample\_fastcgi\_perl\_basic](https://github.com/kyoro/fluxflex_sample_fastcgi_perl_basic)
- Blosxom [https://github.com/mattn/fluxflex-blosxom](https://github.com/mattn/fluxflex-blosxom)
- Simple PSGI app [https://github.com/mattn/fluxflex-plack-min](https://github.com/mattn/fluxflex-plack-min)
- PSGI streaming [https://github.com/hiratara/fluxflex-psgi-streaming](https://github.com/hiratara/fluxflex-psgi-streaming)
- Catalyst [https://github.com/kyoro/fluxflex\_sample\_perl\_catalyst](https://github.com/kyoro/fluxflex_sample_perl_catalyst)

## What am I using?

Unfortunately none of them. My recent interests lie in real-time web
applications that require WebSocket proxying. Available platforms are built on
Nginx+uWSGI or Apache+FastCGI technologies and are not suitable without
additional patching, hacking etc.

I hope this will be changed soon.

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 80:

    Non-ASCII character seen before =encoding in '├──'. Assuming UTF-8
