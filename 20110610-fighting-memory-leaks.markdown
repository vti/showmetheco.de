Title: Fighting memory leaks
Tags: Perl, memory leaks
Comments: no

When developing an async app in Perl it is fairly easy to introduce a memory
leak in many anonymous subroutines. When I faced this issue in PocketIO
(thanks to jegade, fearless enough to run it in production) I tried
a few modules on CPAN like [Devel::Cycle](https://metacpan.org/pod/Devel::Cycle), [Devel::LeakTrace](https://metacpan.org/pod/Devel::LeakTrace) and others. But
it didn't work for me.

[cut]

So what I did is very simple and I hope there is a better way.

First all `DESTROY` methods were implemented in all classes:

    sub DESTROY {
        warn "I am destroyed";
    }

And then a wonderful [Devel::FindRef](https://metacpan.org/pod/Devel::FindRef) module was used (prints all the
references to your variable) for every object that wasn't destroyed.

I wish there was a way to automatically implement all `DESTROY` methods with
debugging info and some general registry of all created objects. Maybe there is
already a module on CPAN and I couldn't find it?
