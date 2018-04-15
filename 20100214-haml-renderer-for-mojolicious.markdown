Title: Haml renderer for Mojolicious
Tags: Perl, Mojolicious, Haml, renderer
Comments: no

After writing [Text::Haml](https://metacpan.org/pod/Text::Haml) which is a Haml implementation in Perl the next logical step was to write a renderer for [Mojolicious](https://metacpan.org/pod/Mojolicious). So here it is [http://github.com/vti/mojox-renderer-haml](http://github.com/vti/mojox-renderer-haml). Besides a renderer itself there is also a [Mojolicious::Plugin::HamlRenderer](https://metacpan.org/pod/Mojolicious::Plugin::HamlRenderer) module that makes it really easy to plug in a new template handler.

    # Load HAML renderer
    $self->plugin('haml_renderer');

And then you can write a new template `template.html.haml` in Haml:

    !!!
    %html
      %body
        #main
          %h2 Hello, world!

And you will get:

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.
    w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html>
      <body>
        <div id='main'>
          <h2>Hello, world!</h2>
        </div>
      </body>
    </html>
