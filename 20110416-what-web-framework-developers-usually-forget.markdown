Title: What Web Framework developers usually forget
Tags: Perl, web, security
Comments: no

There are a lot of web frameworks in the Perl world. A lot of them differ in
features, design, code quality etc. Some of them are popular, others
undeservedly forgotten. One can start writing a new framework because he doesn't
know about the others, doesn't like them or just because he can. We can argue
about the necessity or stupidity of wheel reinventing, about the needed
features, coding standards, even about the object systems, but what every
developer should agree upon is a web security. Below are advices on writing your
framework with security in mind.

[cut]

## Code quality and the best practices

Write clean, maintanable and understandable code. The easier it is to
understand, the easier it is to spot a bug, logic error or a security hole.

Follow the best Perl practices, read PBP, use [Perl::Critic](https://metacpan.org/pod/Perl::Critic).

## Known pitfalls

Read OWASP Developers Guide
[https://www.owasp.org/index.php/Category:OWASP\_Guide\_Project](https://www.owasp.org/index.php/Category:OWASP_Guide_Project) and similar
papers, follow their recommendations.

Read [perlsec](https://metacpan.org/pod/perlsec), google for `Perl security`.

## Code reuse

Try to use widespread Perl modules that surely already have faced security
issues and have dealt with them. Try not to reinvent the wheel unless it is
absolutely necessary and you know what you're doing.

## Code audit and review

Constantly audit your code and patches that other send to you. Listen to what
people are saying, do not ignore security concerns.

## Testing

Make testing an important part of your project, but don't trust it too much
(tests that pass are bad tests, since they don't reveal any errors, but just
make you feel good). Write not only unit tests, but also functional and live
tests. Test your application in production by yourself.

## Good security principles

Encourage your users to take security into account (like: don't run this pile of
crap as root), force them to validate any input data and so on.

## Vulnarebility scanners

Constantly use security scanners when you release your software. It is easy to
do, doesn't take a lot of time and can be free. Below are the open source
scanners I recommend:

nikto [http://cirt.net/nikto2](http://cirt.net/nikto2)

skipfish [http://code.google.com/p/skipfish/](http://code.google.com/p/skipfish/)

w3af [http://w3af.sourceforge.net/](http://w3af.sourceforge.net/)

## Security as a process

Don't forgret that security is a constant process. It never ends, security bugs
can overlap and hide for years, new feature can introduce new problems. You
don't have to be a security expert to write a secure enough software.

## Anything else?

Thanks for reading. I hope to hear advices from you too!
