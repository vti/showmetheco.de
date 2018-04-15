Title: Perl and friends books publications statistics
Tags: Perl, Ruby, Python, PHP, books

When I tried to learn another scripting language I was looking for the books
(eh, the first thing that comes to my mind). I was searching and searching and
then got curious on how many books were published during the last 10 years in
the languages that kinda share the same field. So I called Ruby, Python and PHP
friends and started building statistics.

[cut] See the graphs

It may not be the most correct one, since I searched only via amazon books using
their ecommerce API.

First I tried using [Net::Amazon](https://metacpan.org/pod/Net::Amazon) but it didn't worked very well for paging, so
I just wrote a simple "get the job done" script. I cheated a bit, because
I removed everything that was about the web frameworks (Catalyst, Django, Rails,
Sinatra and so on). Then prepared data for `gnuplot`.

<div>
    <img src="/images/perl-and-friends-books-publications.png" />
</div>

And btw you can check it yourself, here is the script
[https://gist.github.com/3048049](https://gist.github.com/3048049).

I am working on a small project that somehow uses this data, so stay tuned for
the announcement!
