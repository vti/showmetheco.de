Title: Accessors are dangerous
Tags: Perl, accessors

So commonly used accessors (setters/getters) I think are dangerous when
overused or used without a caution. Here is a quick list why.

[cut]

- Too many accessors are a sign of a class that does too much or is highly coupled
- Too much information is revealed about the class
- Accessors that have the same name for setting and getting variables may be confused with normal methods since they don't state clearly if they are `read only`

        $invoice->price(100); # Nothing happens
        $invoice->price;      # Anything but 100

- Encourage breaking of `Tell Dont Ask` and `Law of Demeter` principles

        if ($invoice->price > 100) {
            $invoice->pay;
        }

- Require too much of configuration when more advanced behaviour is needed

        has 'book' => sub {

            # Setter
            if (@_) {
                my $value = shift;

                if ($value =~ m/.../) {
                    ...
                }

                return;
            }

            # Getter
        };

- Trying to be universal add a lot of checks that are not always needed
- Too many ways to do it

        Class::MethodMaker
        Object::Tiny
        Spiffy
        Class::Spiffy
        accessors
        Object::Tiny
        Rubyish::Attribute 
        Class::Accessor
        Class::Accessor::Fast
        Class::Accessor::Complex
        Class::Accessor::Constructor
        Class::Accessor::Classy
        Class::Accessor::Lite 
        Class::XSAccessor 
        Moose
        Mouse
        Moo
        Mo

    and

        Mojo::Base::XS # yeah, baby, yeah!

    See also [App::Benchmark::Accessors](https://metacpan.org/pod/App::Benchmark::Accessors).

- Are slow and require optimization magic (literally)

    See [Class::XSAccessor](https://metacpan.org/pod/Class::XSAccessor)

    For me it's not difficult to write 3-5 methods that set/get variables while
    keeping encapsulation and ease of modification (though I must admit I have CPAN
    modules that do not follow this approach, but the time will come!).
