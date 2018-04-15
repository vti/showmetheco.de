Title: Contracts in mop
Tags: Perl, mop, contracts

While trying out [mop](https://metacpan.org/pod/mop) I decided to implement contracts, or basically a simple
type checking system.

[cut]

The type checking itself is done via [Type::Tiny](https://metacpan.org/pod/Type::Tiny). In [mop](https://metacpan.org/pod/mop) it is implemented
using method roles, which is very very handy. The code looks like:

    use mop;
    use mopx::contracts;

    use Types::Standard -types;

    class MyClass {
        has $!foo is expected(Int);

        method add ($add) is expected(Int), ensured(Int) {
            $!foo + $add
        }
    }

So `mopx::contracts` exports `expected` and `ensured`. As you can see this
works on attributes too. `ensured` is for checking the return values, thus
adding even more robustness to our code.

In case you wonder what happens when the class is inherited, all the types are
inherited too, there is no need to repeat them. For example, an abstract class
(or interface) can look like:

    class MyAbstractClass {
        method do_something is ensured(Int)
    }

    class MyClass extends MyAbstractClass {
    }

And `MyClass` inherits all the checking.

It is possible to overwrite the type checking now, but I will modify it so you
can only overwrite the types without breaking Liskov substitution principle. For
example:

    class MyAbstractClass {
        method do_something is expected(Int), ensured(Int)
    }

    class MyClass extends MyAbstractClass {
        method do_something is expected(Number), ensured(PositiveInt)
    }

So we can only weaken preconditions and strengthen postconditions. But this is
not yet implemented :) (**UPDATE**: implemented!)

If you're interested you can follow the project at
[http://github.com/vti/mopx-contracts](http://github.com/vti/mopx-contracts).

I want to thank guys who are making [mop](https://metacpan.org/pod/mop) happen and for their help on the
channel, especially `doy`, who helped with initial implementation.
