Title: TDD Best Practices in Perl
Tags: Perl, TDD

In this article I've collected the best practices of TDD
([Test Driven development](http://en.wikipedia.org/wiki/Test-driven_development))
that help me in my work. I brought them together for the future reference,
updates, sharing and discussion.

[cut]

## Obvious TDD advantages

- Interface is created "automatically"
- Only really needable features are implemented
- System is developed by small steps
- It forces to write modular code
- Design errors are recognized on the early stages
- Module works
- Safe refactoring is possible

## Obvious TDD disadvantages

- Same errors may be left in tests and code
- False sense of security
- More stuff to maintain
- Brittle tests cause failing cases when code is merely changed

Most of the disadvantages can be eliminated by _following_ the best practices.

## Best practices

### Follow the cycle

Follow the cycle of TDD: red, green, refactor. This actually means: 

- write a test before the actual code and run it to make sure it doesn't pass
- write the actual code and make sure the test passes
- refactor written code

### Keep tests readable and clean

Tests are code too. The more readable and clean they are, the easier to maintain
(fix, refactor, move around) they are. If you don't keep your tests clean they
will begin to rot and will create more harm than good.

### Test cases should not depend on each other

Make sure your tests are independent. That means they have independent data
setup, independent state and no side effects. This way you can change or run
a test without affecting others. And you can be sure what you're actually
testing.

### Group your tests or, even better, use classes

This actually means you should group your tests to their own scope. In Perl you
can either use subtests of [Test::More](https://metacpan.org/pod/Test::More) or methods in [Test::Unit](https://metacpan.org/pod/Test::Unit) or
[Test::Class](https://metacpan.org/pod/Test::Class). By using classes where test cases are just normal methods you
can use all the power of OOP (inheritance, encapsulation, polymorphism,
composition etc).

    use Test::More;

    my $foo = new_ok('Foo');
    is($foo->bar, '123');

    done_testing;

Becomes:

    use Test::More;

    subtest 'instance is returned' => sub {
        new_ok('Foo');
    };

    subtest 'default value is correct' => sub {
        my $foo = Foo->new;

        is($foo->bar, '123');
    };

    done_testing;

### One test -- one assert

Testing only one thing at a time makes tests less brittle, more readable and
independent. By following this practice you won't create test cases that are
named as 'simple', 'general' or 'testing everything'. All the tests will have
specific purpose.

    sub sum_the_arguments : Test {
        my $self = shift;

        my $object = ObjectToTest->new;

        my $sum = $object->sum(1, 2);

        is($sum, 3);
    }

Read more about [One Assertion Per Test](http://www.artima.com/weblogs/viewpost.jsp?thread=35578).

### 3A, AAA, Arrange-Act-Assert

You should have three easily distinguishable blocks:

- Arrange all necessary data
- Act on the tested method or object
- Assert that the returned result is what we expect

    sub revert_the_string : Test {
        my $self = shift;

        # Arrange
        my $object = ObjectToTest->new;

        # Act
        my $reverted = $object->revert('abc');

        # Assert
        is($reverted, 'cba');
    }

This makes test case more readable. It is easy to see how the object is
prepared, what method is called and what is the result.

Read more about [Arrange Act Assert](http://www.c2.com/cgi/wiki?ArrangeActAssert).

### Test behaviour rather than implementation to eliminate brittle tests

When tests are too specific or reveal too much information about object
implementation they become brittle. When implementation changes after the first
refactoring tests break and have to be fixed.

Make sure you test the behaviour and not the implementation.

So instead of:

    eval { $object->die_hard };
    is "$@", "We died here for the good reason. Error 42";

Test:

    eval { $object->die_hard };
    like "$@", qr/\s* Error \s+ 42$/xms;

Read more about
[Test for Required Behavior, not Incidental Behavior](http://commons.oreilly.com/wiki/index.php/Test%5Ffor%5FRequired%5FBehavior%2C%5Fnot%5FIncidental%5FBehavior).

### Do not overuse mocks

While mocking is a very useful thing when you don't have the real objects it
could create some problems. Mock implementation can be equal to the real
implementation so there is no benefit. Mocks do not report errors when real
class interface changes.

It is always good to replace mocks with real objects unless they are really
simple and straightforward.

**Update:** See [Discussion with Christian Walde](http://showmetheco.de/articles/2011/11/tdd-best-practices-in-perl.html#comment-356566270)
for an alternative.

### Put object creation into factory methods

If you use inheritable tests put test objects creation into factory methods.
This will remove code duplication and will allow polymorphism.

    sub return_current_time_by_default {
        my $self = shift;

        my $time = $self->_build_time(foo => 'bar');

        is($time->now, time);
    }

    sub _build_time {
        my $self = shift;

        return MyTime->new(@_);
    }

### Move common fixures into Object Mother or Test Data Builders

When you have many tests you have probably duplicate code that creates the same
test data for the different tests. In order to remove duplication you can use
Object Mother or Test Data Builders patterns.

Object Mother is a big factory class that provides all kinds of objects you need
for testing. For example:

    sub dog_catches_cat : Test {
        my $self = shift;

        my $dog = ObjectMother->createDog(name => 'dog');
        my $cat = ObjectMother->createCat(name => 'cat');

        my $catches = $dog->catches($cat);

        ok($catches);
    }

Read more about [ObjectMother](http://martinfowler.com/bliki/ObjectMother.html).

Test Data Builder is another approach when Object Mother becomes bloated and
hard to maintain. This way you create a Builder that builds needed object with
needed data for every test.

    sub dog_catches_cat : Test {
        my $self = shift;

        my $dog = DogBuilder->build(name = 'dog');
        my $cat = CatBuilder->build(name => 'cat');

        my $catches = $dog->catches($cat);

        ok($catches);
    }

Read more about [Test Data Builders: an alternative to the Object Mother pattern](http://www.natpryce.com/articles/000714.html).

### Readability over abstraction

Too much of abstraction in tests make them harder too read. It is better to
introduce some code duplication to make them clearer. Do not hide your test
data too far, it should be easy to find and understand.

### Unit testing is not enough

TDD is not just about [Unit Testing](http://en.wikipedia.org/wiki/Unit_testing).
You can increase the coverage of your code by writing black box tests using the
same cycle of red, green, refactor. This idea is used in BDD
([Behavioural Driven Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development))
and combines best practices of TDD with functional testing. But this is a topic
for another article.

## Recommended testing modules

[Test::More](https://metacpan.org/pod/Test::More) is not the best choice unless you are grouping your tests. Tests
with bunch of `ok` and `is`  scattered throughout the code are not readable
and maintainable. They depend on each other, they create side effects and can
test the same thing over and over again. Lots of code duplication, same
initialization code can be found in different files. It is probably possible to
follow the best practices while using [Test::More](https://metacpan.org/pod/Test::More) but there are better tools.

[Test::Unit](https://metacpan.org/pod/Test::Unit) or [Test::Class](https://metacpan.org/pod/Test::Class) are a good choice for writing OOP testing
classes. Although [Test::Unit](https://metacpan.org/pod/Test::Unit) looks somewhat different from the familiar
[Test::Builder](https://metacpan.org/pod/Test::Builder) interface.

Lots of BDD modules appeared lately, like [Test::More::Behaviour](https://metacpan.org/pod/Test::More::Behaviour), [Test::Spec](https://metacpan.org/pod/Test::Spec),
[Test::Expectation](https://metacpan.org/pod/Test::Expectation), [Test::Behaviour::Spec](https://metacpan.org/pod/Test::Behaviour::Spec) and [courgette.pl](https://metacpan.org/pod/courgette.pl),
[Test::BDD::Cucumber](https://metacpan.org/pod/Test::BDD::Cucumber), [Test::Pcuke](https://metacpan.org/pod/Test::Pcuke). But they only add sugar syntax to your
tests, and sometimes this sugar makes tests less readable. TDD is more about the
methodology and approach than about the tools. You can achieve the same results just
by following best practices and using "normal" testing modules.
