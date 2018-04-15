Title: Yet another try/catch module
Tags: Perl

[Try::Tiny](https://metacpan.org/pod/Try::Tiny) is nice. But it's just an eval wrapper. Most of the time I have to
catch exceptions by specifying their `isa`s. And I want to receive an object in
every `catch` callback, without overriding the global `$SIG{__DIE__}`. I like
[Error::Simple](https://metacpan.org/pod/Error::Simple), but it's not supported and not recommended. That's why I
decided to reimplement it.

[cut] See the code

Meet [http://github.com/vti/error-tiny](http://github.com/vti/error-tiny)! Instead of writing this:

    try {
        dangerous();
    }
    catch {
        my $e = $_;

        if (blessed($e) && $e->isa('MyCustomException')) {
        }
        else {
        }
    };

you can write this:

    use Error::Tiny;

    try {
        dangerous();
    }
    catch MyCustomException with {
        my $e = shift;

        ...everything whose parent is MyCustomException...
    }
    catch {
        my $e = shift;

        ...everything else goes here...
    };

There is no `finally`, because, well, you can always right the code after the
try/catch block anyway. Maybe I'm missing something here, let me know (Jesse
Luehrs pointed out that it's needed when `catch` block itself throws an
exception, in this case `finally` will still be executed).

    use Error::Tiny;

    try {
    }
    catch {
    };

    ... finally ...

If you don't need to specify an exception's class just use the simple form:

    use Error::Tiny;

    try {
        dangerous();
    }
    catch {
        my $e = shift;

        ...everything else goes here...
    };

You can rethrow exceptions too:

    use Error::Tiny;

    try {
        dangerous();
    }
    catch {
        my $e = shift;

        $e->rethrow;
    };

You will always get an object in the catch block. No need to check if it's a
blessed reference or anything like that. And there is no need for
`$SIG{__DIE__}`!

    use Error::Tiny;

    try {
        die 'my string';
    }
    catch {
        my $e = shift;

        # $e is the default Error::Tiny::Exception object!
    };

`Error::Tiny::Exception` is a lightweight base exception class. It is easy to
throw an exception:

    Error::Tiny::Exception->throw('error');

File and line are (hopefully) correctly saved and exception itself is
correctly stringified.

    $e->message;
    $e->file;
    $e->line

    my $errr_message =  "$e";

There are some warnings though:

- an indirect syntax is used for the `catch .. with` part
- a `with` keyword doesn't play nice with [Moose](https://metacpan.org/pod/Moose)

Yes, I know about [Try::Tiny::ByClass](https://metacpan.org/pod/Try::Tiny::ByClass) and [Try::Tiny::SmartCatch](https://metacpan.org/pod/Try::Tiny::SmartCatch). But
their syntax is not very readable to me. But of course, TIMTOWTDI!
