Title: SQL::Composer - mapping SQL from Perl and back
Tags: Perl, sql

SQL::Composer is yet another SQL mapper. But unlike others it does something
very useful. It allows you to not only build an SQL from a Perl structure, but
when getting the data from database to map it back to the usable Perl
structure.

[cut]

Some times you don't need an ORM, but you would like to have an SQL builder
that hides escaping, allows you to specify joins and returns a usable Perl
structure when getting data from the database. SQL::Composer does exactly that.

For several years I have been waiting for a module that would do this, but
unfortunately all the modules I have tried lack the join support at least. And
I use joins a lot. Also they are tightly coupled with other ORMs or require a
lot of manual parsing and stuff. So I have written my own module and several
people found it handy. So here it goes.

Let's start from the most advanced example where the best part of SQL::Composer
is shown. For example we have Review -> Book -> Author tables. And we want
to fetch a Review with all the related information.

```
my $expr = SQL::Composer::Select->new(
    from    => 'review',
    columns => ['text'],
    join    => [
        {
            source  => 'book',
            columns => ['title'],
            on      => [id => {-col => 'review.book_id'}],
            join    => [
                {
                    source  => 'author',
                    columns => ['name'],
                    on      => [id => {-col => 'book.author_id'}]
                }
            ]
        }
    ],
    where => [id => 1]
);
```
Let's generate SQL:

```
my $sql = $expr->to_sql;

# SELECT
#     `review`.`text`,
#     `book`.`title`,
#     `author`.`name`
# FROM `review`
# JOIN `book` ON `book`.`id` = `review`.`book_id`
# JOIN `author` ON `author`.`id` = `book`.`author_id`
# WHERE `review`.`id` = ?

```
And get the bind values:

```
my @bind = $expr->to_bind;
```

After getting an ARRAYREF from DBI we will get a correctly mapped HASHREF that
can either be used as is or can be converted into an object very easily (simple
and tiny Perl value objects without additional functionality).

```
my $sth = $dbh->prepare($sql);
my $rv  = $sth->execute(@bind);

my $rows = $sth->fetchall_arrayref;
my $row_object = $select->from_rows($rows)->[0];

my $objects = $expr->from_rows($rows);
```

If `$rows` is something like:

```
[['Good', 'Perl programming', 'YAPH']]);
```

then we will get:

```
# [
#     {
#         'book' => {
#             'title'  => 'Perl programming',
#             'author' => {
#                 'name' => 'YAPH'
#             }
#         },
#         'text' => 'Good'
#     }
# ];
```

This structure is intuitive and maps perfecly to hash based Perl objects. If
you do not want joins be embedded into other joins, just don't specify them as
that:

```
my $expr = SQL::Composer::Select->new(
    from    => 'review',
    columns => ['text'],
    join    => [
        {
            source  => 'book',
            columns => ['title'],
            on      => [id => {-col => 'review.book_id'}],
        },
        {
            source  => 'author',
            columns => ['name'],
            on      => [id => {-col => 'book.author_id'}]
        }
    ],
    where => [id => 1]
);
```

And we will get:

```
# [
#     {
#         'book' => {
#             'title' => 'Perl programming',
#         },
#         'author' => {
#             'name' => 'YAPH'
#         }
#         'text' => 'Good'
#     }
# ];
```

Among joins and data mapping SQL::Composer supports all expected expressions
you would find in other SQL builders.

```
[foo => 'bar']              => "`foo` = ?",   ['bar']
[foo => { '!= ' => 'bar' }] => "`foo` != ?",  ['bar']
[foo => {-col => 'bar'}]    => "`foo` = bar", []
```

Also it supports sometimes hard to build sql expressions like:

```
[created => {'>' => \['ADD_DATE(NOW(), INTERVAL ? SECOND', 10]}]

# "created >= ADD_DATE(NOW(), INTERVAL ? SECOND)", [10]
```

I have been using SQL::Composer in
[ObjectDB](http://metacpan.org/module/ObjectDB) for quite a while, and it works
perfectly well in production with real world data and problems (of course if
you like me try to reduce logic in SQL queries). The main focus is on joins and
related objects, since when you have a "good enough" normalization you have a
lot of joins. But as I stated earlier some people may use SQL::Composer
directly since it is very handly and removes a lot of boilerplate from your
Perl code.

One can say that writing raw SQL is more readable. Yes, that is true for very
complex queries. But most of the time you need some kind of automation,
arguments injections and so on and you end up implementing some kind of an SQL
builder yourself. Also it is easier to map data from the database when it is
structurely presented rather than parsing SQL which is not very easy and not
very portable.
