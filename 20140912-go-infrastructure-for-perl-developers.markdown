Title: Go infrastructure for Perl developers
Tags: Go, Golang, Perl

So I have decided to look into the Go language a bit closer. Last time I ended
doing just a simple tutorial. This time I decided to rewrite this blog engine
(<http://github.com/vti/Twist>) in Go. The result can be found on GitHub
(<http://github.com/vti/twigo>). There is no really a need to describe the
language learning issues in this blog post, but I wanted to share a comfortable
infrastructure that I researched while trying to learn Go.

[cut]

## Installing Go

Installing Go is easy. But you end up messing with various environmental
variables like `GOPATH` and `GOROOT` and so on. I had no luck in setting up an
environment that would work as I expected. But this is very easy with
[gvm](https://github.com/moovweb/gvm). It is something like a `perlbrew` or
`plenv`. This way you can switch between different versions on Go without
messing with the system one. All the dependencies are also installed in your
home directory without any problems.

# Documentation

You can always google your question, but it's a good idea to be able to read
the documentation locally. This is what `godoc` is for. Installing it is simple:

    go get code.google.com/p/go.tools/cmd/godoc

## Testing

Testing is built in into Go, which is very good. Unfortunately it could be a bit
tedios. So if you want instead of this:

    if got != expected
        t.Errorf("got = %v, want %v", got, expected)
    }

write:

    assert.Equal(t, expected, got)

get the [testify](https://github.com/stretchr/testify) package. Also notice that
it follows `expected, got` sequence instead of `got, expected` that we are used
to from `Test::Simple` and friends.

## Continious testing

Writing code, building and testing can be tyring. This is why I was very happy
when I found [GoConvey](http://goconvey.co), which is a utility with a web
interface that detects changes in your code, compiles it and tests it. It
utilizes desktop notifications, so you always know what happened without
checking your browser.

If you install `cover`:

    go get code.google.com/p/go.tools/cmd/cover

You will get a coverage percentages and a link to the html report (which is very
easy to understand btw).

## Checking your code

If you want something like `Perl::Critic` try the `vet` command:

    go get code.google.com/p/go.tools/cmd/vet

And then the simple run inside of your application:

    go vet

will print all the issues (as it thinks :) you have in your code.

## Pretty printing

The embedded `fmt` package and `Println` function for instance can print complex
Go lang structures, but if you want something more advanced like `Data::Dumper`
or `Data::Printer` try [pretty](http://github.com/kr/pretty).

## Command-line option parsing

There are many packages that do command-line option parsing. I personally needed
something easy, so I went with [docopt.go](https://github.com/docopt/docopt.go).

       usage := `twigo

    Usage:
      twigo serve --conf=<conf> --listen=<listen>
      twigo -h | --help

    Options:
      -h --help         Show this screen.
      --conf=<conf>     Path to configuration file (conf.json).
      --listen=<listen> Listen options (:8080).`

       arguments, err := docopt.Parse(usage, nil, true, "twigo", false)
       if err != nil {
           log.Fatal("error parsing command-line options:", err)
       }

There is a `Docopt` package for Perl also.

## Auto imports

It could be very frustrating to add all the imports manually. For example you
want to debug a piece of code by printing several variables, you have to add
`fmt` package at the top of you Go file. You can't leave it if you stop
debugging, since Go will not compile telling you that the package is imported
but not used.  But there is a way to automate this. Try instead of `gofmt` (the
default `Perl::Tidy` of Go) `goimports`:

    go get code.google.com/p/go.tools/cmd/goimports

## Dependencies

For managing dependencies I tried
[goop](https://github.com/nitrous-io/goop) which is something like `Carton`.

## Deploying

Deploying Go programs is not as easy as copying them to the remote machine,
since the program has to be compiled. Fortunately cross compiling is very easy
in Go. The [goxc](https://github.com/laher/goxc) package takes care of that.
After installing it:

    go get github.com/laher/goxc

Just compile the needed architecture. For example my server is i386 linux,
I run:

    goxc -arch="386 amd64" -bc="linux" -os="linux"

And you will get a tarball with all the compiled files. `goxc` has a configuration
file where you can put all the options, including additional files to pack with
your tarball. It also creates a `.deb` package, though I haven't tried
installing it myself.

As for running the Go web application on the server, I start it behind an
`nginx`, controlled via a `supervisord`. My config looks something like:

    [program:showmethecode]
    command = path/to/twigo serve --conf path/to/config.yml --listen 127.0.0.1:8080
    directory = path/to/twigo/
    user = vti
    autostart = true
    autorestart = true

## Vim integration

I just copied the syntax scheme from the Golang website
<https://golang.org/misc/vim/syntax/go.vim>. There is
a [vim-go](https://github.com/fatih/vim-go) package that promises a full
IDE-like Go integration, but I personally don't like those kind of plugins. But
you may find it interesting.

I have also added several hotkeys for code formatting:

    " Go
    au BufRead,BufNewFile *.go setfiletype go
    nnoremap <silent> ,gt :%!goimports <cr>
    vnoremap <silent> ,gt :!goimports <cr>

Why `,gt`? Well, I am used to `,pt` which runs `Perl::Tidy` :)

## Final notes

As for the language itself I found it very productive and pragmatic. Just like
the Perl itself :) I spent more time finding the right tools and packages for
the task, since there are already too many of them. So I hope this article will
help you save some time.

My code is probably not very idiomatic and is more like a baby Go, so I do not
recommend learning from it, instead read the source code of the standard
library.

Have fun!
