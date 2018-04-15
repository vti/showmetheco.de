Title: Throwing and catching exceptions in Perl
Tags: Perl, exceptions

While many people still check every method's return value, parse die's string
messages and think that exceptions should be avoided at all cost, I will try to
convert all exceptions to objects, create the base exception class and catch
them with simple `eval` without using CPAN (blasphemy!).

[cut]

## Throwing

So we want this to behave the same:

    die 'error';

    die MyException->new;

And we do not want to write this every time (from `perldoc die`):

    use Scalar::Util "blessed";
    eval { ... ; die Some::Module::Exception->new( FOO => "bar" ) };
    if (my $ev_err = $@) {
        if (blessed($ev_err) && $ev_err->isa("Some::Module::Exception")) {
            # handle Some::Module::Exception
        }
        else {
            # handle all other possible exceptions
        }
    }

We need to set up our own `$SIG{__DIE__}` handler:

    use Scalar::Util qw(blessed);
    $SIG{__DIE__} = sub {
        my ($e) = @_;

        return unless $^S;

        if (!blessed($e)) {
            $e =~ s/ at .*? line .*?\.//;
            chomp $e;
            $e = MyException->new(message => $e, caller => [caller]);
        }

        CORE::die($e);
    }

The magic `$^S` variable tells us to skip `eval` parsing phase.

We parse exception text because we still have to do it, in 2012!

`chomp` is because we don't want this:

    something went wrong
     at myscript.pl line 42.

Also we can save `caller` information for later use.

## Exception class

    package MyException::Base;

    use strict;
    use warnings;

    use overload
      '""'     => sub { $_[0]->as_string },
      'bool'   => sub {1},
      fallback => 1;

    sub new {
        ...
    }

    sub as_string {
        my $self = shift;

        return sprintf("%s at %s line %s.",
            $self->{message}, $self->{path}, $self->{line});
    }

We can add also `throw`, `rethrow`, `does` and other methods for convenience.

It is important to overload `""` and `bool` because we want this to work:

    print $e;

    # and

    if ($e) {
        # we had an exception
    }

## Catching

We can use [Try::Tiny](https://metacpan.org/pod/Try::Tiny) of course, but doing it with `eval` is as easy (and
`return` works as expected!):

    eval {
        some_function();
        1;
    } || do {
        my $e = $@;

        if ($e->isa('MyException::FileNotFound')) {
            ...
        }
        else {
            ...
        }
    };

Assigning `$@` to a local variable is important since sometimes `$@` behaves
strangely (should ask a Perl guru).

## See also

Of course you can use [Error](https://metacpan.org/pod/Error) (Jakub Narębski points out that it's not
recommended and you should look for [Exception::Class](https://metacpan.org/pod/Exception::Class)) and similar modules
from [CPAN](https://metacpan.org/pod/CPAN) instead of writing you own implementation, but sometimes...

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 118:

    Non-ASCII character seen before =encoding in 'Narębski'. Assuming UTF-8
