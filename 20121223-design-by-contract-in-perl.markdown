Title: Design by contract in Perl
Tags: Perl, design, contract

[Design by Contract](http://en.wikipedia.org/wiki/Design_by_contract) is a
programming approach when method calls are checked againts specific
requirements by embedded in the language or implemented as a library functions.
Usually incoming and outgoing values are checked, sometimes it is possible to
check the throwable exceptions. Below are various modules that allow contracts
in Perl.

[cut]

## Introduction

Contracts provide:

- Checking of input and output values, and thrown exceptions
- Conformance to Liskov (L in SOLID) substitution principle
- Interface documentation

If the method call does not match the contract an exception is thrown.

Contracts are the most useful for interfaces or abstract classes when you want
to control whether your implementation follows the same interface and respects
the Liskov substitution principle (basically whenever the base class is used any
child class could be used).

## The problem

For example if we had an abstract template rendering class:

    package AbstractRenderer;
    sub new {...}

    sub render {
        die 'implement me';
    }

We want to make sure that child classes following the same interface and we
don't get something like:

    package TT;
    use base 'AbstractRenderer';
    sub render {
        my $self = shift;
        my ($template, $layout, %vars) = @_;

        return \$rendered;
    }

    package Caml;
    use base 'AbstractRenderer';
    sub render {
        my $self = shift;
        my ($template, %vars) = @_;

        return $rendered;
    }

We would like to have a module that would make sure that child classes follow
the same interface and it should look something like:

    package AbstractRenderer;
    sub new {...}

    # Requires(STRING, %HASH_OF_ANY_VALUES)
    # Ensures(STRING)
    # Throws(Exception::TemplateNotFound)
    sub render {
        my $self = shift;
        my ($template, %vars) = @_;

        ...

        return $rendered;
    }

When trying to find a solution on CPAN I had the following requirements:

- Contract definition should leave class still readable
- No CPU intensive checking
- Could be switched off
- Must be inheritable
- Denies contract changes in child classes
- Must throw understandable errors
- As little magic as possible
- Should look like Perl

## Solutions

I found the following solutions:

### Class::Contract

    use Class::Contract;

    contract {
      inherits 'BaseClass';

      ctor 'new';

      method 'methodname';
        pre  { ... };
          failmsg 'Error message';

        post  { ... };
          failmsg 'Error message';

        impl { ... };
    };

This doesn't look very much like Perl to me. Especially the `ctor` function
makes it look very odd.

### Sub::Contract

    use Sub::Contract qw(contract);

    contract('surface')
        ->in(\&is_integer, \&is_integer)
        ->out(\&is_integer)
        ->enable;

    sub surface {
        # no need to validate arguments anymore!
        # just implement the logic:
        return $_[0] * $_[1];
    }

I find this too verbose.

### Sub::Assert

    assert
        pre     => {
         'parameter larger than one' => '$PARAM[0] >= 1',
        },
        post    => '$VOID or $RETURN <= $PARAM[0]',
        sub     => 'squareroot',
        context => 'novoid',
        action  => 'carp';

Magic.

### Carp::Datum

    use Carp::Datum;

    sub routine {
        DFEATURE my $f_, "optional message";
        my ($a, $b) = @_;
        DREQUIRE $a > $b, "a > b";
        $a += 1; $b += 1;
        DASSERT $a > $b, "ordering a > b preserved";
        my $result = $b - $a;
        DENSURE $result < 0;
        return DVAL $result;
    }

Makes the method unreadable.

### MooseX::Contract

    contract 'add'
        => accepts [ ...type... ]
        => returns void,
        with_context(
            pre => sub {
                ...
            },
            post => assert {
                ...
            }
        );
    sub add {
        my $self = shift;
        my $incr = shift;
        $self->{value} += $incr;
        return;
    }

I do not need a big OOP framework, just the contracts.

## More solutions

None of the previous solutions quite satisfied me, thus I had to roll out my own
:)

### Attribute::Contract

    package AbstractRenderer;
    use Attribute::Contract;

    sub new {...}

    sub render
      :ContractRequires(VALUE(Str), %ANY)
      :ContractEnsures(VALUE(Str))
      :ContractThrows(Exception::TemplateNotFound) {
    }

It satisfies all my requirements, feels natural because of the attributes
utilization, does only one thing and stays simple.

For now the code is only on GitHub [http://github.com/vti/attribute-contract](http://github.com/vti/attribute-contract).
Read more there to get familiar with the API.

## When (not) to use

Contracts are very much suitable for interface checking, but this should not
transform into validation. Think of a contract as a kind of assert. Checking
specifications also must not create any side effects, since they could be turned
off after development and modify the code behaviour.

Contracts are not a substitution for unit testing of course. They solve
a different task. By unit testing we make sure the classes in isolation behave
as we expect them to, do not introduce regressions after refactoring and so on.
By contracts we provide a safety net in runtime during object interactions.

Contracts should not be used on the layer of user data validation, since on this
step we want to follow a defensive programming approach: recover from errors,
create meaningful messages etc. We do not want to throw an exception, then catch
it and return the error to the user. Nonetheless contracts are very suitable for
library modules when you want to fail as soon as possible and as hard as
possible to make the life easier for those who happen to use them.
