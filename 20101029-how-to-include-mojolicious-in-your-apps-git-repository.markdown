Title: How to include Mojolicious in your app's git repository
Tags: Perl, Mojolicious, git
Comments: no

When you are developing an application that you want to deploy really fast and
easy you should consider including [Mojolicious](https://metacpan.org/pod/Mojolicious) files in your repository.

Some of the advantadges of this approach are:

- You don't have to worry about installing Mojolicious
- You don't have to worry about non compatible Mojolicious version installed on the production server
- You always have a Mojolicious version that works with your application
- Creating a tarball that constains everything can be done by git

One of the disadvantages is that it is not easy to setup, but this post is going
to fix that.

[cut]

There are two major ways to include other projects in your git repository:
`submodules` and `subtrees`. We start with the easiest but not the best.

## Submodules

So you have your application under git. For example project's tree looks like
this:

    $ ls
    lib/       log/       public/    script/    t/         templates/

Let's create a `contrib` directory where we will put all 3rd party modules (in
the simplest case only [Mojolicious](https://metacpan.org/pod/Mojolicious)):

    $ mkdir contrib
    $ ls
    contrib/   lib/       log/       public/    script/    t/         templates/

Now we add a `submodule` that will containt [Mojolicious](https://metacpan.org/pod/Mojolicious) repository.

    $ git submodule add http://github.com/kraih/mojo.git contrib/mojo
    Cloning into contrib/mojo...
    ...
    $ ls contrib/mojo
    Changes        LICENSE        MANIFEST.SKIP  Makefile.PL    README.pod
    examples/      lib/           script/        t/

So now we have a separate directory that holds all [Mojolicous](https://metacpan.org/pod/Mojolicous) files. But the
next time your repository is cloned you will have to update submodules too.

    $ git submodule update --init

When updating submodules a normal `git pull` can be used. In case if you have
many submodules you can update them within one command:

    $ git submodule foreach 'git pull'

## Subtrees

In the case of a subtree we inject [Mojolicious](https://metacpan.org/pod/Mojolicious) repository as a subdirectory
that doesn't look any different from other files in your repository.

First we add a remote branch:

    $ git remote add -f mojo http://github.com/kraih/mojo.git
    Updating mojo
    ...
    $ git branch -r
      mojo/master

Then we want to pull [Mojolicious](https://metacpan.org/pod/Mojolicious) as a subdirectory to the project:

    $ git read-tree --prefix=contrib/mojo/ -u mojo/master

Now commit to add the files:

    $ git commit -m 'Added Mojolicious as a subtree'

Every time when you want to update `contrib/mojo` directory run this command:

    $ git merge --squash -s subtree --no-commit mojo/master
    ...
    Squash commit -- not updating HEAD
    Automatic merge went well; stopped before committing as requested

So we import all the changes as one patch and we don't do automatic commit. You
can commit now:

    $ git commit -m 'Updated Mojolicious'

In the case of subtree you don't have to do anything except cloning.
[Mojolicious](https://metacpan.org/pod/Mojolicious) files will just be there.

## SEE ALSO

More on `submodules` [http://book.git-scm.com/5\_submodules.html](http://book.git-scm.com/5_submodules.html)

More on `subtrees` [http://progit.org/book/ch6-7.html](http://progit.org/book/ch6-7.html)
