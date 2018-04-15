Title: Sending email from Mojolicious
Tags: Perl, email, Mojolicious
Comments: no

Ever wondered what is the easiest way to send an email from [Mojolicious](https://metacpan.org/pod/Mojolicious)? Some
simple way that would allow you to use templates and more advanced stuff? There
is already a plugin for that — [Mojolicious::Plugin::Mail](https://metacpan.org/pod/Mojolicious::Plugin::Mail)!

[cut]

[Mojolicious::Plugin::Mail](https://metacpan.org/pod/Mojolicious::Plugin::Mail) fits very naturally. It introduces a new `mail`
template format and handy `mail` and `render_mail` helpers.

If you don't need any special behaviour just load the plugin without any
parameters:

    # Mojolicious::Lite
    plugin 'mail';

    # Mojolicious
    $self->plugin('mail');

For a more controllable and advanced behaviour the following configuration could
be made:

    $self->plugin(
        mail => {
            from     => 'vti@example.com',
            encoding => 'base64',
            how      => 'sendmail',
            howargs  => ['/usr/sbin/sendmail -t'],
        }
    );

Sending an email from a controller is not harder:

    $self->mail(to => 'vti@example.com',
        template => 'controller/action', format => 'mail');

That will grab a `controller/action.mail.ep` template:

    % stash subject => 'Emaling from Mojolicious is fun!';
    Hello, world!

If you are really demanding and want to build a message yourself you can always
use a `mail` helper, that has a [MIME::Lite](https://metacpan.org/pod/MIME::Lite) interface.

    $self->mail(
        to      => 'vti@example.com',
        from    => 'vti@example.com',

        cc      => '..',
        bcc     => '..',

        subject => 'Emailing from Mojolicious is fun!',
        data    => 'Hello, world!',
    );

For attachments, additional headers, encodings and so on see the original
documentation [Mojolicious::Plugin::Mail](https://metacpan.org/pod/Mojolicious::Plugin::Mail).

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 5:

    Non-ASCII character seen before =encoding in '—'. Assuming UTF-8
