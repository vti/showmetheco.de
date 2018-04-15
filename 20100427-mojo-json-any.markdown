Title: Mojo::JSON::Any
Tags: Perl, Mojolicious, JSON
Comments: no

While there is always [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON) out there, sometimes we need more speed.  [JSON::XS](https://metacpan.org/pod/JSON::XS) is the first thing that comes to mind. But they not only have different interfaces, but also work different with encodings.

[cut]

So here is [Mojo::JSON::Any](https://metacpan.org/pod/Mojo::JSON::Any), a tiny wrapper around [JSON::XS](https://metacpan.org/pod/JSON::XS) that has the same interface and works exactly the same as [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON) with encodings. The tests suite is almost identical.

[Mojo::JSON::Any](https://metacpan.org/pod/Mojo::JSON::Any) follows the same phylosophie as all Any modules on CPAN do.  When no [JSON::XS](https://metacpan.org/pod/JSON::XS) is available, [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON) is used. When [JSON::XS](https://metacpan.org/pod/JSON::XS) is available but it use is not desirable `MOJO_JSON` variable can be set to fall back to using [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON). If no variable is set and [JSON::XS](https://metacpan.org/pod/JSON::XS) is available then it is used.

Git repository is on github [http://github.com/vti/mojo-json-any](http://github.com/vti/mojo-json-any).
