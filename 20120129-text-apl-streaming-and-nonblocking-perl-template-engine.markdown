Title: Text::APL - streaming and non-blocking Perl template engine
Tags: Perl, template, streaming, non-blocking

This is yet another template engine. But compared to others it supports
non-blocking (read/write) and streaming output.

[cut]

## Syntax

Syntax is borrowed from the template standards shared among several web
framewoks in different languages. Many developers are already familiar with it:

    <% foo() %> # evaluate code
    % foo()

    <%= $foo %> # insert evaluation result
    %= $foo

    <%== $foo %> # insert evaluation result without escaping
    %== %foo

No new template language is provided, just the old good Perl.

But the interesting part is in getting the source and printing the output.

## Simple example

    $template->render(
        input  => \$input,
        output => \$output,
        vars   => {foo => 'bar'}
    );

## Streaming example

    $template->render(
        input => sub {
            my ($cb) = @_;

            # Call $cb($data) when data is available
            # Call $cb->() on EOF
        },
        output => sub {
            my ($chunk) = @_;

            # Print $chunk to the needed output
            # $chunk is undef when template is fully rendered
        },
        vars => {foo => 'bar'}
    );

## Reader/Writer

Reader and writer can be a subroutine references reading from any source and
writing output to any destination. Sane default implementations for reading from
a string, a file or file handle and writing to the string, a file or a file
handle are also available.

## Parser

Parser can parse not only full templates but chunk by chunk correctly resolving
any ambiguous leftovers. This allows immediate parsing.

This for example works just fine:

    $parser->parse('<% $hello');
    $parser->parse(' %>');

## Compiler

Compiler compiles templates into Perl code but when evaluating does not create
a Perl string that accumulates all the template output, but rather provides
a special `print` function that pushes the content as soon as it's available
(streaming).

The generated Perl code can look like this:

    Hello, <%= $nickname %>!

    # becomes

    __print(q{Hello, });
    __print_escaped(do {$foo});
    __print(q{!});

## Real life example

In the `examples/` directory you can find a runnable (!) example of [Plack](https://metacpan.org/pod/Plack),
[AnyEvent](https://metacpan.org/pod/AnyEvent) and [IO::AIO](https://metacpan.org/pod/IO::AIO) working together. The template is opened and read
asyncronously by [IO::AIO](https://metacpan.org/pod/IO::AIO) and than the output is printed using `chunked`
transfer encoding to the client (using [Plack::Middleware::Chunked](https://metacpan.org/pod/Plack::Middleware::Chunked)).

    use strict;
    use warnings;

    use Plack::Builder;

    use File::Basename ();
    use File::Spec;
    use AnyEvent::Handle;
    use AnyEvent::AIO;
    use IO::AIO;

    use Text::APL;

    my $template       = Text::APL->new;
    my $templates_path = File::Basename::dirname(__FILE__);

    my $app = sub {
        my ($env) = @_;

        return sub {
            my ($respond) = @_;

            my $writer = $respond->([200, ['Content-Type' => 'text/html']]);

            my $handle;

            my $path_to_template =
              File::Spec->rel2abs(
                File::Spec->catfile($templates_path, 'template.apl'));

            aio_open $path_to_template, IO::AIO::O_RDONLY, 0, sub {
                my $fh = shift or die "$!";

                my $reader = sub {
                    my ($cb) = @_;

                    $handle = AnyEvent::Handle->new(
                        fh      => $fh,
                        on_eof  => sub { $cb->() },
                        on_read => sub {
                            my $handle = shift;

                            $handle->push_read(
                                line => sub {
                                    my ($handle, $line) = @_;

                                    $cb->($line);
                                }
                            );
                        },
                        on_error => sub { }
                    );
                };

                my $writer = sub {
                    my ($chunk) = @_;

                    if (defined $chunk) {
                        $writer->write($chunk);
                    }
                    else {
                        $writer->close;
                    }
                };

                $template->render(
                    input  => $reader,
                    output => $writer,
                    vars   => {name => 'vti'}
                );
            };
        };
    };

    builder {
        enable 'Chunked';

        $app;
    };

The source code can be found at [http://github.com/vti/text-apl](http://github.com/vti/text-apl).
