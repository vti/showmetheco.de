Title: Announcing Kritika.IO, a service for analysing your Perl code
Tags: Perl

For the last year and a half I have been doing some code reviewing at work. And I noticed that most of the time the tasks are reopened because of very small things, like text formatting, variable naming and so on. This could've been prevented just by using Perl::Critic for example. Unfortunaly the code base is big, and that would take a lot of time. That's why I have written <https://kritika.io> which makes the analysis incrementally and is damn easy to integrate with GitHub and Travis CI (or anything else actually). The service is in beta right, but i have been using it secretly for the last 5 months anyway. It is useful for me, maybe it can be useful for you too.

[cut] Read more for more info!

## Creating an account

Right now you can create an account using your GitHub... well, account. I have already implemented BitBucket and GitLab support, but decided not to release too many things and first focus on getting one thing right. Later you can login with your username/email if you want.

## Integrating with GitHub

When you import a repository from GitHub Kritika can automatically install a webhook add get a push event. After analysing the source code, Kritika will update commit status on GitHub with the link to the snapshot (this is how I call a scanning report). You will also get an email notification (which can be switched on/off in account settings).

## Integrating with CI services

By installing [Devel::Cover::Report::Kritika](https://metacpan.org/pod/Devel::Cover::Report::Kritika) you can upload test coverage to Kritika. Test coverage reporting still needs some work, but at least the results are saved fully and will not need to be reuploaded later. Upload now and enjoy future features :)

Here is an example for TravisCI `.travis.yml`.

```
language: perl
perl:
    - "5.22"
    - "5.24"
before_install:
    - cpanm -n Devel::Cover::Report::Kritika
install:
    - cpanm -n -q --with-recommends --skip-satisfied --installdeps .
script:
    - perl Build.PL && ./Build build && cover -test -report kritika
```

## Mapping your repository files to rules

Kritika doesn't do anything automatically. I don't know yet if this is good or bad, but you have to 1) group you sources (for example Core files group, test files, documentation files etc) and 2) assign Rules Profiles to the groups.

The rules are mapped in repository configuration using `YAML` format (yes, I am not trying to be original here). Here is an example for a typical Perl application:

```
---
# Group files in your project
# (paths are .gitignore-like)
natures:
  - nature: perl
    paths:
      - lib/**.pm
      - "!**/SkipMe.pm"
  - nature: js
    paths:
      - "public/**.js"
# Assign profiles to every file group
profiles:
  # Analyze Perl files
  - profile: MyPerlProfileTitle
    natures:
        - perl
  # Analyze JavaScript files
  - profile: MyJSProfileTitle
    natures:
        - js
  # Analyze all sources files
  - profile: MyGenericTextProfileTitle
    natures:
        - perl
        - js
```

As you can see this gives a lot of flexibility. For example you can create a special profile for analysing your Perl test files, which can be very different from analysing the core files.

## Seeing what has changed between snapshots

This is the most useful functionality. It gives a possibility to see what actually has been changed in a new commit. For example when doing a Code Review you want to see only new issues with the code.

## Duplications

Duplications are hard to get right. For now they are disabled in Kritika, but they are almost done. Right now I build an AST using PPI and then just search for the code patterns, which means that variable names, spaces etc don't affect the comparison. This turned out to be very effective and soon will be enabled for everybody.

## Roadmap

I have a lot of things in mind. Here is a show list of the features that are going to be implemented next:

- ~~Pull Request support (and report back to GitHub or any other service with comments)~~ **done**
- ~~Org Repositories~~ **done (not only repos, but full organizations accounts)**
- ~~Duplications (as I already said, this is partly done)~~ **done partly**
- ~~More Rules profiles (like #TODO search, file name patterns, etc)~~ **done partly**
- ~~Integration with BitBucket and GitLab (this is partly done, so won't take too long)~~ **done**
- ~~Private Repositories (this already works, because I use Kritika for analysing Kritika :), but disabled for the beta period)~~ **done**
- ~~Kritika Enterprise (install Kritika on your own server)~~ **done**

Let me know if you like it and have any interesting ideas!

Go and sign up for <https://kritika.io>.
