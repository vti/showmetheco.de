Title: Writing Perl code the right way
Tags: perl, the right way

Today we are going to learn how to write a HTTP request dispatcher subroutine in
Perl. Why Perl? Because Perl style is close to HTML style. And HTML is used on
the internets. Below are some useful tips.

[cut]

Use one-letter variables. It saves not only the space on the monitor but also your fingers.

Don't use `use strict` because it breaks code.

Don't use CPAN modules. Just don't.

Code formatting considered harmful. It is easier to spot a bug when code fits into one paragraph.

Don't refactor, clean, decouple your code. If you have to change your code later, you'll have enough time.

No testing. Production is the best way to test things.

Don't use OOP. It is useful only for team work.

CGI is the only true interface. It's new, stable and fast.

Below is an example that follows the best practices:

    sub Request {
    if(lc($ENV{'REQUEST_METHOD'}) eq "get") {$B=$ENV{"QUERY_STRING"};} else {binmode STDIN; $BR=$B=join('',<STDIN>);}
    $bn=$ENV{'CONTENT_TYPE'}; if ($bn =~ /multipart/){$bn=~s/\+/\\+/g; $bn=~s/.*=/--/g; $B=~s/.*?$bn(.*)$bn.*/$1$bn/gs;
    foreach $V (split(/\r*\n$bn/, $B)){$V=~/name=\"(.*?)\"/; $N=$1; if ($V=~/filename=/) {$V=~/filename=\"(.*?)\"/; $FN{$N}=$1;} $V=~s/.*?\r*\n\r*\n//s; utf8::decode($V); $R{$N}=$V;}}
    else {foreach $V (split(/&/,$B)) {($N, $V)=split(/=/,$V); $V=~s/\+/ /g; $V=~s/%([a-fA-F0-9][a-fA-F0-9])/pack("U", hex($1))/eg; utf8::decode($V); $R{$N}=$V;}}
    }

Talking to people on IRC can be dangerous for your brain.
