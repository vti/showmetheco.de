Title: Perltuts - Interactive Perl tutorials
Tags: Perl, tutorials

Let me introduce to you the [http://perltuts.com](http://perltuts.com), a website with interactive
Perl tutorials where you can run the code in your browser!

[cut] How does it work

## Tutorials

The tutorials themselves are just POD documents that are imported into the
database. When the webpage is rendered all the code is transformed into editable
textarea elements with [CodeMirror](http://codemirror.net/). When you click the
`Run` button an ajax call is performed sending the code to the evaluator, which
gives back `stdout`, `stderr` and test results if they were present.

## Evaluator

Evaluator is a simple daemon that runs on a virtual machine, it accepts the
code, builds the Perl package and evals it capturing the output with
[Capture::Tiny](https://metacpan.org/pod/Capture::Tiny). It does some timeout checking and fork limiting of course.

The virtual machine (`qemu` Debian image) allows you to run a real Perl code on
a real machine. This can be used when writing advanced tutorials including IO,
networking, forks etc (I tried [Safe](https://metacpan.org/pod/Safe) but wanted more freedom). It is reset
every hour from a snaphot. 

## Exercises

Exercises are just the code where output after evaluation is run against the
prebuilt [Test::More](https://metacpan.org/pod/Test::More) package. The tests can be as sofisticated as needed.

## Books

You may have noticed there are some Perl books presented. I've collected the
latest editions, so the beginners can pick the up-to-day Perl books easily
(there is an article about the
[books](http://showmetheco.de/articles/2012/7/perl-and-friends-books-publications-statistics.html)
btw).

## Want to get involved?

Sure, see the [Get Involved](http://perltuts.com/get_involved) page!

Maybe you've already seen [http://learnpython.org](http://learnpython.org) or [http://rubymonk.com](http://rubymonk.com).
Perltuts is something like that, but for Perl! Of course it's still in progress,
there is only one basic and not complete tutorial, but I just want to have the
feedback as early as possible.

Let me know what you think!
