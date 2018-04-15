Title: How to avoid Unicode pitfalls in Mojolicious
Tags: perl, mojolicious, unicode
Comments: no

Unicode is hard. Unicode in Perl is even harder, because sometimes Perl is just
too smart. While [Mojolicious](https://metacpan.org/pod/Mojolicious) is a web framework, no wonder why it should
support Unicode really well. But even if [Mojolicious](https://metacpan.org/pod/Mojolicious) tries hard to make
things easy for a developer, one must really understand what's going on behind
the scene.

[cut]

There is a really good documentation on using Unicode in Perl. Raise your hand
if you haven't read `perluniintro`, `perlunicode`, `perlunifaq` or
`perlunitut`. I knew that.

A byte and a character is not the same thing. Sometimes they have the same
length (all ASCII characters), but most of the time (other Unicode characters)
their length is different. For example, the dollar character `$` is 1 byte
long, but euro `€` is 3. In Perl we want to work with characters.

For decoding bytes to characters and encoding characters to bytes we use
[Encode](https://metacpan.org/pod/Encode) module. I kept always forgetting what is encoding and what is
decoding. But if you think for a minute and imagine this evil world that lives
in bytes, and you want to take something from it and bring it to Perl, you have
to decode it, so you can understand it. And otherwise when you have to send
something you have to encode your stuff.

There is also `utf8` pragma. Everybody keeps thinking that if you use it, you
don't have any problems, that Perl will do everything for you. But the problem
is that `utf8` pragma must be used ONLY when you have unicode characters in
your module or perl file. It doesn't take care of file handles or databases.

Back to the evil world. It lives in bytes. When using [Mojolicious](https://metacpan.org/pod/Mojolicious) client's
request comes in bytes and [Mojolicious](https://metacpan.org/pod/Mojolicious) answers in bytes too. To make things
easy for a developer everything inside is automatically decoded into Perl
characters. This way you have parameters, captures and stash values all in Perl
characters. You can run regexes on them, sort them, count their length etc.

When a developer bypasses the [Mojolicious](https://metacpan.org/pod/Mojolicious) mechanism of decoding, like opens a
file or opens a database connection, he has to be sure to decode octets to Perl
characters (i.e., using ':utf8' for file handles, [Mojo::ByteStream](https://metacpan.org/pod/Mojo::ByteStream) for
strings, or database modules internal methods).

Below are some examples that show what is right and what is wrong to do in
[Mojolicious](https://metacpan.org/pod/Mojolicious).

## EXAMPLES

### [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON)

`decode` accepts a sequence of octets (not characters!), calculates a correct
encoding (UTF-8, UTF-16 etc) and returns a Perl arrayref or hashref with
correctly decoded values.

    use utf8;

    # Wrong
    $json->decode('{"foo":"ü"}');

    # Right
    $json->decode(b('{"foo":"ü"}')->encode('UTF-8'));

`encode` accepts a Perl structure with correctly decoded values and returns
octets (not characters) in UTF-8.

    use utf8;

    # Wrong
    $json->encode(foo => 'ü') . 'ü';

    # Right
    b($json->encode(foo => 'ü'))->decode('UTF-8') . 'ü';

### [Mojo::DOM](https://metacpan.org/pod/Mojo::DOM)

`parse` accepts Perl characters OR octets, detecting and decoding them
automatically.

    use utf8;

    # Right
    $dom->parse(b('ü')->encode('UTF-8'));

    # Right too
    $dom->parse('ü');

And returns Perl characters and thus must be encoded before printing it to the
console for example.

    # Wrong
    print $dom->at('a')->text;

    # Right
    print b($dom->at('a')->text)->encode('UTF-8');

### [ojo](https://metacpan.org/pod/ojo)

When crawling a web site, don't forget to `encode` the result before printing
it to console. [Mojo::ByteStream](https://metacpan.org/pod/Mojo::ByteStream)'s `say` method not only adds a newline
character, but also automatically encodes the data.

    # Wrong
    perl -Mojo -e 'print g("mojolicio.us")->dom->at("title")->text'

    # Right
    perl -Mojo -e 'b(g("mojolicio.us")->dom->at("title")->text)->say'

## SEE ALSO

[http://www.slideshare.net/Penfold/perl-and-unicode](http://www.slideshare.net/Penfold/perl-and-unicode)

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 18:

    Non-ASCII character seen before =encoding in 'C<€>'. Assuming UTF-8
