Title: My date bug or not so consistent consistency
Tags: Perl

So the other day I was writing a method that checks if a user provided date was
a valid one. What could be easier? I quickly wrote code, tested it and pushed to
production. The thing is that on production system the test woudn't pass. What
the f...antastic debugging opportunity!

[cut]

The function was like this:

    print is_date_ok(1962, 1, 2), "\n";

    sub is_date_ok {
        my ($year, $month, $day) = @_;

        eval {
            Time::Local::timegm(0, 0, 0, $day, $month - 1, $year - 1900);
            1;
        } || do {
            0;
        };
    }

So I started debugging. Added warning to the eval block and got this on
production:

    Cannot handle date (0, 0, 0, 2, 0, 2060)

Ok. So something's wrong with the year calculation, but I remember reading this
in [Time::Local](https://metacpan.org/pod/Time::Local) documentation:

    [...]the year should be specified in a form consistent with
    "localtime()", i.e. the offset from 1900[...]

but there was more:

    [...]Years in the range 0..99 are interpreted as shorthand for years
    in the rolling "current century," defined as 50 years on either side
    of the current year. Thus, today, in 1999, 0 would refer to 2000,
    and 45 to 2045, but 55 would refer to 1955. Twenty years from now,
    55 would instead refer to 2055. This is messy, but matches the way
    people currently think about two digit dates.  Whenever possible,
    use an absolute four digit year instead.[...]

This is messy? Yes, it is!

So the dates before 62 (1962 - 1900) are interpreted as 2062 (2012 + 50) and are
too big. Ok, I got confused because this is very common:

    my $current_year = (localtime(time))[5] + 1900;

Not a big deal, let's pass full four-digit year:

    Time::Local::timegm(0, 0, 0, $day, $month - 1, $year);

And it works!

Wait a minute. Why then tests would pass on my machine? A not so quick
investigation revealed that I had perl 5.14 and production -- perl 5.10. So
apparently the date was wrong (2062 instead of 1962), but the [Time::Local](https://metacpan.org/pod/Time::Local) on
5.14 was happy with it.

The lesson? Don't trust your tests! Test not only that the date is valid but
test also if the date is the actual date you're validating. And reread the
documentation.
