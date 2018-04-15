Title: Attribute::Contract now uses Type::Tiny
Tags: Perl, contract

I became very interested in [Type::Tiny](https://metacpan.org/pod/Type::Tiny) as soon as I saw Toby's first post
(yes I like modules that do one thing and do not depend on half of the CPAN).
And then I thought why not to actually use it. And this is why
[Attribute::Contract](https://metacpan.org/pod/Attribute::Contract) now uses [Type::Tiny](https://metacpan.org/pod/Type::Tiny)!

[cut]

    package MyClass;

    use AttributeContract -types => [qw/ClassName Object Str/];

    sub new :ContractRequires(ClassName) :ContractEnsures(Object) {
        ...

        return $self;
    }

    sub method :ContractRequires(Object, Str) :ContractEnsures(Str) {
        my $self = shift;
        my ($string) = @_;

        return $another_string;
    }

This to me looks much better than (sorry Toby :):

    state $check = compile(Str, Str, slurpy ArrayRef[Num]);
    my ($sort_code, $account_number, $monies) = $check->(@_);

Moreover [Attribute::Contract](https://metacpan.org/pod/Attribute::Contract) makes sure that the rules are inheritable, so in
a child class you do nothing when the needed method is not overwritten or just
`use` the module if it is.

    package MyClassChild;
    use base 'MyClass';
    use AttributeContract;

    sub method {
        ...
    }

Also, inlike with [Method::Signatures](https://metacpan.org/pod/Method::Signatures), it is possible the check the return
values too. See more details in my previous article
[http://showmetheco.de/articles/2012/12/design-by-contract-in-perl.html](http://showmetheco.de/articles/2012/12/design-by-contract-in-perl.html).
