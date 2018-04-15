Title: Mustache-like template engines for Perl
Tags: Perl, template, mustache
Comments: no

I am used to [Mojo::Template](https://metacpan.org/pod/Mojo::Template). But I noticed that I keep improving the template
files by substituting a lot of html by Perl code or helper functions that
generate the part of the template. Below I am trying a new approach in making my
templates easier to read and maintain.

[cut]

So here are typical parts of the template:

    % if ($author) {
    <span id="author"><%= author %></span>
    % }

    % foreach my $author (@$authors) {
        <%= $author %><br />
    % }

Maybe to someone it looks just fine, but for me there is too much logic. I don't
see html anymore, it is just plain Perl. Moreover when trying to simplify a
template I try to hide various calculations into the helper functions, like
here:

    % foreach my $meta (@{config('meta')}) {
            <meta
    % for my $key (keys %$meta) {
    <%== "$key=\"$meta->{$key}\" " %>
    % }
    />
    % }

It becomes:

    <%= meta %>

And I thought that I am trying to eliminate logic from my templates anyway, why
then just not try a logic-less template engine? The first example becomes:

    {{#author}}
    <span id="author">{{author}}</span>
    {{/author}}

    {{#authors}}
        {{author}}<br />
    {{/authors}}

Looks much better to me. I don't have to worry whether I have a scalar, an array
reference or a subroutine. It iterates over my data automatically doing what I
meant.

Probably the most popular logic-less template engine is
mustache [http://mustache.github.com](http://mustache.github.com) and it is available for various languages
including Perl ([https://github.com/pvande/Template-Mustache](https://github.com/pvande/Template-Mustache)).

And I couldn't resist from implementing one more version too
[http://github.com/vti/text-caml](http://github.com/vti/text-caml) :)
