Title: Test::More inside of Test::More
Tags: Perl, testing

While working on [http://perltuts.com](http://perltuts.com) I had to write a test runner what would
test tutorial exercises. It would create a Perl package that uses [Test::More](https://metacpan.org/pod/Test::More)
than evaluate it, catch the output and return to the caller. This was easy. But
than as a good Perl guy I had to test it, actually I had to write a test before
writing an implementation, but that's another story! So... I wanted to test it
with a [Test::Class](https://metacpan.org/pod/Test::Class) (uses [Test::Builder](https://metacpan.org/pod/Test::Builder) internally). Needless to say it
didn't work, because [Test::More](https://metacpan.org/pod/Test::More) is an evil singleton! I had to deploy soon so
I had to do something. And I did. I did a very dirty thing and it worked.

[cut]

So there is a TestRunner module. Usage is pretty simple:

    my $test_runner = Perltuts::TestRunner->new;
    $test_runner->run($code, $eval_result, $test);

Where `$code` is initial source code of an exercise, `$eval_result` is
a result of evaling it and `$test` is the actual test.

Now let's come back to the pain I had with a TestRunner. In order to isolate
[Test::More](https://metacpan.org/pod/Test::More) to its own environment the following steps were made:

Reset the singleton (this was needed even before testing, because once compiled
the exercise's test wasn't working with the following evaluations):

    local \$Test::Builder::Test = undef;

Catch the TAP output (oh-my-eyes!):

    my \$OUTPUT = '';
    no warnings 'redefine';
    local *Test::Builder::_print = sub { \$OUTPUT .= \$_[1] };

Check if the test was successful (after lurking in the source code):

    return (Test::More->builder->is_passing ? 'success' : 'failed', \$OUTPUT);

Will all the above manipulations plus a manual `stderr` capturing (yes, I know
about [Capture::Tiny](https://metacpan.org/pod/Capture::Tiny) but it didn't work, maybe because [Test::More](https://metacpan.org/pod/Test::More) and
friends do some really weird stuff) the final module looks like this:

    package Perltuts::TestRunner;

    use strict;
    use warnings;

    sub new {
        my $class = shift;

        my $self = {@_};
        bless $self, $class;

        return $self;
    }

    sub run {
        my $self = shift;
        my ($code, $result, $content) = @_;

        $result->{stdout} = '' unless defined $result->{stdout};
        $result->{stderr} = '' unless defined $result->{stderr};

        my $test_code = <<"EOF";
    package Test;
    sub {
        use strict;
        use warnings;

        open OLDERR, ">&", STDERR;
        open STDERR, ">>", '/dev/null';

        use Test::Builder;
        local \$Test::Builder::Test = undef;

        use Test::More;

        my \$OUTPUT = '';
        no warnings 'redefine';
        local *Test::Builder::_print = sub { \$OUTPUT .= \$_[1] };

        my \$code   = q{$code};
        my \$stdout = q{$result->{stdout}};
        my \$stderr = q{$result->{stderr}};

        $content;

        done_testing;

        open STDERR, ">&", OLDERR;

        return (Test::More->builder->is_passing ? 'success' : 'failed', \$OUTPUT);
    }
    EOF

        my $sub = eval $test_code or die $@;
        my ($status, $output) = $sub->();

        return {status => $status, output => $output};
    }

    1;

And the test looks like this:

    sub return_success_when_test_passes : Test {
        my $self = shift;

        my $runner = $self->_build_runner;

        my $result =
          $runner->run('', {stdout => '1'}, q{is($stdout, 1, 'Should be 1')});

        is_deeply($result,
            {status => 'success', output => "ok 1 - Should be 1\n1..1\n"});
    }

Yes, monkey-patching is evil. Monkey-patching of a private method is even more
evil. But this is the best I could do. If you know a better solution, please,
tell me!
