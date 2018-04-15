Title: Perl routes dispatching comparison
Tags: Perl, routes, comparison

There are several separate distributions on CPAN that implement routes
dispatching. Below is my attempt in comparing [Router::Simple](https://metacpan.org/pod/Router::Simple),
[Path::Router](https://metacpan.org/pod/Path::Router), [HTTP::Router](https://metacpan.org/pod/HTTP::Router), [Routes::Tiny](https://metacpan.org/pod/Routes::Tiny) and [Path::Dispatcher](https://metacpan.org/pod/Path::Dispatcher).

[cut]

    +-------------------+-------------+----------+----------+----------+---------~
    | Name              | Constraints | Optional | Grouping | Globbing | To path ~
    +-------------------+-------------+----------+----------+----------+---------~
    | Router::Simple    |      +      |    -     |    -     |    +     |    -    ~
    | Path::Router      |      +      |    +     |    -     |    -     |    +    ~
    | HTTP::Router      |      +      |    -     |    -     |    -     |    +    ~
    | Routes::Tiny      |      +      |    +     |    +     |    +     |    +    ~
    | Path::Dispatcher  |      +      |    +     |    +     |    +     |    -    ~
    +-------------------+-------------+----------+----------+----------+---------~

    ~-------------------+-----------+------+
    ~ Name              | Subroutes | PSGI |
    ~-------------------+-----------+------+
    ~ Router::Simple    |     +     |  +   |
    ~ Path::Router      |     +     |  +*  |
    ~ HTTP::Router      |     +     |  -   |
    ~ Routes::Tiny      |     -     |  -   |
    ~ Path::Dispatcher  |     +     |  +   |
    ~-------------------+-----------+------+

\* via [Plack::App::Path::Router](https://metacpan.org/pod/Plack::App::Path::Router) (thanks to Arun Prasaad)

# Explanations

## Constraints

Ability to limit a capture to a specific regexp. For example, we want to match
`/articles/1` but not `/article/foo`, so we add a constraint like `\d+`.

## Optional

Ability to add optional captures. For example, we want to match `/blog/2010/01`
and `/blog/2010` without specifying an additional route.

## Grouping

Ability to capture not just between slashes. For example, we want to match
`/hello-user`, capturing `hello` and `user`.

## Globbing

Ability to capture path using globbing. For example, we want to match
`/book/title/that/constains/slashes` without worring what goes after `title`.

## To path

Ability to build a path from a route. For example, we don't want to specify
paths directly in the templates, so we can use something like

    build_path('article', id => 1)

and get `/articles/1`.

## Subroutes

Ability to mount subroutes on a specific route. For example, we want to add
routes that all start with `/admin/`.

## PSGI support

Easy integration into `PSGI` frameworks. Ability to match against `PSGI`
environment.

# Disclaimer

If you found any error, please correct me.
