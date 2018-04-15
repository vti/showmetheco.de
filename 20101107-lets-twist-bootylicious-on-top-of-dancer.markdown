Title: Let's Twist: Bootylicious on top of Dancer
Tags: Perl, Bootylicious, Dancer
Comments: no

It doesn't matter what framework you use, but what does matter is how good your
model is. By model I mean M in MVC. There are plenty ways to parse HTTP, to
dispatch an url, to render a template, to server a static file. But there is
only one way to:

    my $booty = Bootylicious->new;
    $booty->get_articles_by_tag('foo');

With a good model you can switch web frameworks, template engines and more. So
to prove it the Twist was born.

[cut] What is Twist?

Twist is Bootylicious (originally written in [Mojolicious](https://metacpan.org/pod/Mojolicious)) on top of
[Dancer](https://metacpan.org/pod/Dancer). Well, it is just another way to **view** and **control** Bootylicious.
In some ways it is harder, in some ways it is easier, but it is definetely
possible.

The good model separation also protects you from unexpected changes in
underlying layers. As long as you control the model, your application stays
stable.

# SEE ALSO

Bootylicious source code [http://github.com/vti/bootylicious](http://github.com/vti/bootylicious).

Twist source code [http://github.com/vti/twist](http://github.com/vti/twist).
