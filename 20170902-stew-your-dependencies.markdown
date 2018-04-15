Title: Stew your dependencies
Tags: Perl

So you want to ship your Perl application but cannot install anything on the destination host? Or maybe you want your
own perl version? Or you depend on lots of system libraries of specific versions? Or you ship to several different Linux
distributions? You just want to update a single archive, unpack it and run it without worrying about compatibility? Try
[stew](https://github.com/vti/stew).

[cut] Yes, yes, yes!

**TL;DR**

Check out [example application](https://github.com/vti/stew-example-app) and
[example repository](https://github.com/vti/stew-example-repo).

**Long version**

So at my previous work we had to ship our application to customers with different distributions from Debian and Ubuntu
to SuSe and CentOS. Installing any packages was not an option. Compiling was not an option either. We wanted to make
distribution and installation as simple as possible and as less painful as possible.

We started by compiling by hand all the libraries our software needed, putting it into the `local/` directory inside the
application and shipping the tarball to the customer. Pretty soon this became very tedious process, since we were
shipping CPAN modules too and they were updating pretty often.

So I ended up developing my own package manager with dependencies and versions that would produce a shippable self
contained application. Welcome [stew](https://github.com/vti/stew)!

Basically you would create a repository with sources and stewfiles. Stewfiles are files that contain instructions on how
to build a specific package, for example here is how we build relocatable perl:

    ---
    name: perl
    version: 5.22.1
    package: ${name}-${version}
    sources:
        - ${package}.tar.gz
    depends:
        - patch
        - less
    prepare:
        - tar xzf '${package}.tar.gz'
        - cd ${package}
    build:
        - cd '${package}'
        - >
            ./Configure
            -Duselargefiles
            -Duse64bitint
            -Dprefix=${PREFIX}
            -Dman1dir=none
            -Dman3dir=none
            -Uuseshrplib -Duserelocatableinc
            -Dvendorprefix=.../..
            -Accflags=-DPERLIO_BUFSIZ=32768
            -Uafs
            -Ud_csh
            -Ud_ualarm
            -Uusesfio
            -Uusenm
            -Ui_libutil
            -des
        - make
    install:
        - cd '${package}'
        - make install

As you can see it is possible to declare other dependencies and customize the build process. After stew builds the
stewfile it creates a binary package, in case of perl and Debian it would be `perl_5.22.1_linux-debian-9-x86_64.tar.gz`.
This binary package can be cached and during the next installation there is no need to compile anything.

Common repository layout:

    dist/
        linux-debian-9/
            x86_64/
                perl_5.22.1_linux-debian-9-x86_64.tar.gz
                ...
        linux-centos-7/
        linux-suse-11/
    stew/
        perl_5.22.1.stew
        ...
    src/
        perl-5.22.1.tar.gz
        ...

Inside of our application we place a list of stew packages we require. The application layout can look like this:

    my_app/
        bin/stew
        ...
        local/
        ...
        stewfile

Where `bin/stew` is a fatpacked stew version, `local/` is the directory where we are going to install all the
dependencies, and `stewfile` looks like this:

    perl
    libmodule-build-perl
    libmodule-runtime-perl
    libb-hooks-parser-perl

    openssl
    libgd
    ...

As you can see we build not only perl and CPAN modules, but also our own `openssl` and `libgd`.

Bootstrapping is as simple as:

    bin/stew install .

After running the bootstrap our `local/` directory will look smth like this:

    bin/
    include/
    lib/
        perl5/
            5.22.1/
            site_perl/
            vendor_perl/
    libexec/
    share/
    ssl/
    stew.snapshot

In order to run the programs from the `local/` directory we use `stew exec` command:

    bin/stew exec perl -V

For more details and options take a look at the [stew repository](https://github.com/vti/stew).

Of course creating stewfiles by hand for CPAN modules would be a crazy idea, that is why there is
a [cpan2stew](https://github.com/vti/stew/blob/master/script/cpan2stew) script that will generate the final stewfile
with all the dependencies.

So what are you waiting for? Stew your dependencies!
