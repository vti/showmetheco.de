Title: Top 10 reasons why we won't use your CPAN distribution
Tags: Perl, CPAN
Comments: no

Here is a list I've built based on various answers from people on Twitter, IRC
and other sources. Thanks guys!

We won't use your CPAN distribution if:

[cut]

# 10. You don't care about users

Why did you even uploaded it on CPAN if you don't even care if anybody is going
to use it? Why did you write your contacts if you don't ever answer questions?

# 9. It is not actively maintained

Several open bug reports for years don't add any trust. Not maintained code
begins to rot, stays in the past and finally dies.

# 8. It reinvents and does not improve other distribution

There is nothing wrong with wheel reinventing as long as it's an evolution, the
next turn of the helix. As long as it adds something new, as long as it provides
an alternative, not a clone. The better implementation will survive.

# 7. API is not comprehensible

Non intuitive API forces us to open distribution's documentation hundreds times a
day. We have to write an Adaptor just to make it readable, understandable and
maintanable.

# 6. It is not backward compatible

Of course it is impossible to be 100% backward compatible, but nobody expects
that. What drives people crazy is the ridiculous API changes across minor versions
that break every application built on top of the module in several places
without obvious reasons and comprehensible explanations.

# 5. It does too little

Too little does not mean more than 1 line of code. It means that there is
obvious and bound to a particular case code that is not abstract enough for a
separate distribution.

# 4. It does too much

Developers don't need complete solutions, they need flexible tools to build
their own solutions. We use modules because they save us some work, not because
they replace our work with a highly coupled brick that doesn't move.

# 3. The code is unreadable or is poorly written

It is always easier to rewrite a module from scratch than understand how the
unreadable code works. With unreadable code you can't even appreciate the work
that could be saved by using it. You see just a bunch of ascii characters that
were typed by a monkey. Actually unreadable and poorly written code is the same
thing. Why would you ever trust a module that you don't understand?

# 2. There are no good tests

Tests are more important than a readable code, because you can refactor it
without a fear to break anything. Of course it is not always true, expecially
when the test suite isn't complete. But it gives you a good starting point
before writing your own tests, which has to be done anyway to make sure you
understand the API and it works as expected in a particular case in a specific
environment.

Tests are also a good source of examples.

# 1. It doesn't work

By `it doesn't work` I mean it doesn't work in general, completely not workable
in a real life. This may sound obvious, but everything else is not important
unless the module works. It doesn't matter how many tests there are (even worse
if they pass), how clean and precise the code is. If it doesn't work why did you
put it online?

If you don't check it in a real life and are deceived by a feeling that you have
1k tests and they all pass, that's going to end badly. Fallacious confidence
will lead to a situations when you trust your tests more than users' bug reports
that come from the real life applications, multiple server environments and
operating systems. Don't get fooled.

# Conclusion

Are there any other things that stop you from using a specific distribution on
CPAN?
