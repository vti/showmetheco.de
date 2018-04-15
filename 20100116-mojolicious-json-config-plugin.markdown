Title: Mojolicious JsonConfig plugin
Tags: mojo, mojolicious, perl, plugin
Comments: no

So now the JSON configuration that i used in [Bootylicious](https://metacpan.org/pod/Bootylicious) and Pastelicious [http://github.com/vti/pastelicious](http://github.com/vti/pastelicious) is available as [Mojolicious](https://metacpan.org/pod/Mojolicious) plugin and was added by Sebastian Riedel to the core [Mojo](https://metacpan.org/pod/Mojo) code. The source code is available at github [http://github.com/kraih/mojo/blob/master/lib/Mojolicious/Plugin/JsonConfig.pm](http://github.com/kraih/mojo/blob/master/lib/Mojolicious/Plugin/JsonConfig.pm). 

[cut]

Configuration file is in JSON format but first is preprocessed by
[Mojo::Template](https://metacpan.org/pod/Mojo::Template) that gives an access to the main application object via
`$app` variable or `app` helper.

An example could look like this:

    {
        "foo"       : "bar",
        "music_dir" : "<%= app->home->rel_dir('music') %>"
    }

Plugin is loaded as usual, but also returns parsed config so it is possible to use it during `startup` stage (to load other plugins, do some general one-time configuration etc).

    # Mojolicious::Lite
    my $config = plugin('json_config');

    # Mojolicious
    my $config = $self->plugin('json_config');

Plugin is configurable. Options are `file`, `default` and `stash_key`.

If `file` is not specified it will try `myapp.conf` file (where myapp is application's name) under the application's home directory. If file is specified as an absolute path it will be handled as expected.

Option `default` is used when default configuration is required. When parsing is done configuration will be merged.

After configuration is preprocessed and parsed it is put into the `stash`. The stash key is configurable through option `stash_key`. It is `config` by default.

The full featured example could look like this:

    my $config = plugin json_config => {
        file      => '/etc/myapp.json',
        stash_key => 'conf',
        default   => {title => 'My App'}
    };
