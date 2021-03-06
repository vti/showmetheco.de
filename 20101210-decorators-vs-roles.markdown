Title: Decorators vs Roles
Tags: Perl, decorator, role, inheritance
Comments: no

Inheritance is bad. But we can replace inheritance with various other technics
like decorators or roles. Decorator is a pattern where you wrap another class
adding additional methods and pointing other methods to the original class. Role
combines mixins (where you inject new methods to the class) and interfaces
(where you require a class to implement specific methods). I am going to show
the differences between them, their weak and strong sides.

[cut]

## Problem

We have a class `Window` and we have two classes `WindowWithVerticalBar` and
`WindowWithHorizontalBar`. And now we want to have
`WindowWithVerticalAndHorizontalBar`. Using inheritance we have to use
multiple inheritance, and there is nothing wrong with it for now. But what if
later we want to have `WindowWithVerticalAndHorizontalAndDiagonalBar`,
`WindowWithVerticalAndDiagonalBar` and so on... The solution is to use
decorator or role. Let's look at them a bit closer.

Here is a general `Window` class. We will use it in both implementations.

    package Window;

    sub new { my $class = shift; bless {@_} => $class }

    # Simple get/set attribute
    sub size {
        my $self = shift;

        return $self->{size} unless @_;

        $self->{size} = $_[0];

        return $self;
    }

    1;

## Decorators

When implementing a decorator we create an object that has all the needed
methods being added step by step. We just want to make sure that original
methods are still accessible via the new object.

### WindowWithVerticalBar class

    package WindowWithVerticalBar;

    sub new {
        my $class = shift;
        my $window = shift;

        bless {window => $window} => $class;    # Save the window object
    }

    sub size { shift->{window}->size(@_) }      # Point size to the window
                                                # object

    sub vertical_bar {
        my $self = shift;

        return $self->{vertical_bar} if @_;

        $self->{vertical_bar} = $_[0];

        return $self;
    }

    1;

### WindowWithHorizontalBar class

Looks exactly like `WindowWithVerticalBar` with a replaced name and
`vertical_bar` by `horizontal_bar`.

### WindowWithVerticalAndHorizontalBar class

    my $window = Window->new(size => 42);

    # Add vertical_bar method
    my $window_with_v_bars = WindowWithVerticalBar->new($window);

    # Add horizontal_bar method
    my $window_with_v_and_h_bars =
      WindowWithHorizontalBar->new($window_with_v_bars);

This way we can chain any objects in no particular order, mixing them as we
like, adding new methods on every stage. Moreover we can create more classes in
the future and modify the final object behaviour without touching any previous
classes.

Of course we can refactor and create a separate general purpose decorator class
(e.g. by using `AUTOLOAD`), but it's good enough for this example.

## Roles

When implementing a role we create a class that is able to inject methods into
the class where it is loaded. This can be done by altering Perl's symbol table
(known also as `stash`).

### WindowWithVerticalBar class

    package WindowWithVerticalBar;

    sub import {
        my $package = caller;

        no strict 'refs';
        *{$package . '::vertical_bar'} = \&vertical_bar;
    }

    sub vertical_bar {
        my $self = shift;

        return $self->{vertical_bar} if @_;

        $self->{vertical_bar} = $_[0];

        return $self;
    }

    1;

### WindowWithHorizontalBar class

Looks exactly like `WindowWithVerticalBar` with a replaced name and
`vertical_bar` by `horizontal_bar`.

### WindowWithVerticalAndHorizontalBar class

Here we have to create a separate class that will combine `vertical_bar` and
`horizontal_bar` methods:

    package WindowWithVerticalAndHorizontalBar;

    use base 'Window';

    use WindowWithVerticalBar;   # Injects vertical_bar
    use WindowWithHorizontalBar; # Injects horizontal_bar

    1;

And now create the object:

    my $window_with_v_and_h_bars =
      WindowWithVerticalAndHorizontalBar->new(size => 42);

This way we can inject as many methods as we like. Unfortunately we can't use
`WindowWithVerticalBar` directly and have to create additional class.

## Conclusion

Decorators have less magic, you don't need to change symbol table, don't have to
worry about redefined subroutines and don't have to use inheritance at all. On
the other side roles don't create additional objects.

Choose whatever suits better.
