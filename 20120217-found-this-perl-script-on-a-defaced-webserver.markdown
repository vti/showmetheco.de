Title: Found this Perl script on a defaced webserver
Tags: Perl, wtf

So the other day one of our web servers was defaced and I found a Perl script
that was used to clean up the logs and other leftovers. I opened it in vim and
was like "What da ...". Let's go through the code with me.

[cut]

    #!/usr/bin/perl
    use strict;

No `use warnings` but that'll work too. So far so good.

But then 170 lines of code without newlines, subroutines or proper indentation.

A big fat `if` with `exit`s:

    if (...) {
        # 157 lines of code
        exit;
    }
    else {
        ...
        exit;
    }
    EOF

What? Who writes code like this?

    system "echo -e \"\\033[01;37mDefacing all homepages ...\"\n";

Why on earth use `system` and `echo` for the output even with escape sequences?

Then 14 files are checked for existence and are removed. With 14 `if`s like:

    if (-e '/some/file') {
        system 'rm -rf /some/file';
        # output file was erased
    }
    else {
        # output file does not exist
    }

And then some of them are removed for the second time (just in case) with:

    system 'find / -name 'some file' -exec rm -rf {} \;';

I am so disappointed in this Perl script. So disappointed. And btw no
`.bash_history` was erased.
