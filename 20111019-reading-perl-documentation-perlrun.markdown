Title: Reading Perl documentation: perlrun
Tags: perl, documentation

Recently I reread `perlrun` and found amazing things I didn't know about.
Maybe this can be interesting for somebody else too.

[cut]

## Tracing Perl script

Something like `set -x` for bash scripts:

    PERLDB_OPTS="NonStop=1 AutoTrace=1 frame=2" perl -dS ./myscript.pl

## Turn on Unicode

When printing Unicode text on the command line, you don't have to encode it,
just use `-C` switch:

    perl -C -Mutf8 -e 'print "привет\n"'

## `-e` can be used multiple times

    perl -e 'print "hello";' -e 'print "\n"'

## Copy files with perl

If you can't use `cp` you can use perl!

    perl -p -i'/some/file/path/*' -e 1 file1 file2 file3

## Extract Perl script

You received an email and there is a Perl script somewhere, you don't have to
extract it yourself. Let perl do it. By using `-x` switch perl will try to find
Perl script and will run it.

    Hi,

    here is a Perl script I wrote. Try it.

    #!/usr/bin/env perl

    print "Hello world\n";

And then:

    perl -x email.txt

Reading documentation is useful :)

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 19:

    Non-ASCII character seen before =encoding in '"привет\\n"''. Assuming UTF-8
