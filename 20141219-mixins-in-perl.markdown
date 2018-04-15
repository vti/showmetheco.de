Title: Mixins in Perl
Tags: perl, mixin

If you want to use mixins in Perl you don't have to install anything or play
with symbol table yourself. It's right there, in the core.

[cut]

Mixins are basically not creatable classes, their roles is to embed methods into
your class. It is seen as an alternative to multiple inheritance and is
something like Roles--.

To embed methods you can use plain old simple `Exporter`!

```
package MyMixin
use parent 'Exporter';

our @EXPORT_OK = qw(log);

sub log {
    my $self = shift;

    # Yes, this works too!
    $self->some_internal_method;

    say @_;
}

package SomeClassElsewhere;

use MyMixin 'log';

sub do_stuff {
    my $self = shift;

    $self->log('it is here!');
}

sub some_internal_method {
    my $self = shift;
}
```

KISSing is nice!
