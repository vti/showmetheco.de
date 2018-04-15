Title: Connecting GitHub, Kritika.io and TravisCI
Tags: Perl

Let's implement the real world scenario: every time we push to GitHub we want to run perl critic on
[Kritika](https://kritika.io) rules and tests with coverage on TravisCI. Then we want to merge these results together in
one place to be able to say: yes, this is a good commit (or a pull request)! Connecting all three services is an easy
3-steps process (5mins approximately).

[cut] Keep on reading

## Preparations

You must have a [GitHub](https://github.com) account, duh! With it we will login later to [Kritika](https://kritika.io)
and [TravisCI](https://travis-ci.org).

## Pick a Perl repository

Pick any pick repository, it doesn't have to be even your own one. You can do a fork of a popular repository.

## Kritika configuration

Register (or Login) to [Kritika](https://kritika.io) using GitHub account:

<img src="/static/images/connect-github-kritika-and-travisci/sign-up-github.png" />

### Import your repository

Click on Import from Github button

<img src="/static/images/connect-github-kritika-and-travisci/kritika-import-repo.png" />

And then choose your repository

<img src="/static/images/connect-github-kritika-and-travisci/kritika-select-repo.png" />

Create a new Coverage profile

<img src="/static/images/connect-github-kritika-and-travisci/kritika-create-profile.png" />

Go back to repository and click Edit

<img src="/static/images/connect-github-kritika-and-travisci/kritika-edit-repo.png" />

Add profile to the profiles section

<img src="/static/images/connect-github-kritika-and-travisci/kritika-config-profile.png" />

That's it! Done configuring Kritika.

## TravisCI configuration

Add to the repository `travis.yml` with the following configuration:

    language: perl
    perl:
        - "5.24"
    before_install:
        - cpanm -n Devel::Cover::Report::Kritika
    install:
        - cpanm -n -q --with-recommends --skip-satisfied --installdeps .
    script:
        - perl Build.PL && ./Build build && cover -test -report kritika

This will run tests with coverage and report it to the Kritika.

Enable repository in TravisCI configuration

<img src="/static/images/connect-github-kritika-and-travisci/travisci-enable-repo.png" />

That's it! Done configuring TravisCI.

## Pushing

Now create a new commit and push to your repository.

```
echo 'my changes' > README
git commit -am 'testing ci'
git push origin master
```

## Viewing results

Now if you go back to [Kritika](https://kritika.io), you can see that a new Snapshot was creating and it was analyzed.
After couple minutes when TravisCI will finish running the tests with coverage the Snapshot will receive the report and
it will look something likes this (coverage may vary :):

<img src="/static/images/connect-github-kritika-and-travisci/kritika-coverage.png" />

You can view the coverage file by file too:

<img src="/static/images/connect-github-kritika-and-travisci/kritika-file-coverage.png" />

The GitHub commits will also have a link to the [Kritika](https://kritika.io) report:

<img src="/static/images/connect-github-kritika-and-travisci/kritika-github-snapshot.png" />

Have fun testing and reporting! If you have any questions you are more than welcome to ask!
