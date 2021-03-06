Title: Using JQuery and AJAX in Mojolicious
Tags: Perl, Mojolicious, JavaScript, JSON, AJAX, JQuery
Comments: no

I don't really know how to program in JavaScript and that's why I chose JQuery to do all the job with AJAX. Let's see how short and easy the solution in [Mojolicious::Lite](https://metacpan.org/pod/Mojolicious::Lite) could be.

[cut]

To distinguish between normal requests and AJAX requests I check for `X-Request-With` header that contains `XMLHttpRequest` string (but remember that this header is set up by JQuery, if you use another JavaScript library read its documentation for setting headers). This way we can make our app to work even when user doesn't have or hasn't allowed any JavaScript. Also, when there is no JavaScript all links should look like normal links (AJAX links usually have dashed border at the bottom) and also should work (so no '#' links by default). We can do that with JQuery and that is why at first I wrote a normal application that doesn't know anything about JavaScript and then using JQuery added everything else.

To run this demo save it on your disk and run `perl ajax-demo.pl`, default [Mojolicious](https://metacpan.org/pod/Mojolicious) server will be started on `3000` port. So in your browser type `http://localhost:3000/` and you're done. If you see a dashed link there is JavaScript in your browser and AJAX request will be made and you will see "Hello from AJAX!" response, otherwise you will get "Hello from Mojolicious!".

JQuery library is loaded via Google's content distribution network. So if don't have internet connection while testing the script, download JQuery ([http://jquery.com/](http://jquery.com/)) yourself, put it into `public/` directory and change `script` tag in demo template.

And here is the code. Having `render_json` controller method in [Mojolicious](https://metacpan.org/pod/Mojolicious) makes it easy to implement.

    #!/usr/bin/perl

    use Mojolicious::Lite;

    get '/' => 'index';

    get '/ajax' => sub {
        my $self = shift;

        my $header = $self->req->headers->header('X-Requested-With');

        # AJAX request
        if ($header && $header eq 'XMLHttpRequest') {
            $self->render_json({answer => "Hello from AJAX!"});
        }

        # Normal request
        else {
            $self->render(answer => 'Hello from Mojolicious!');
        }
    } => 'index';

    shagadelic('daemon');

    __DATA__

    @@ index.html.ep
    <html>
        <head>
            <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.4.1/jquery.min.js"></script>
            <script type="text/javascript">
                $(document).ready(function() {

                    // Replace .no-ajax class with .ajax to get dashed borders
                    $(".no-ajax").removeClass("no-ajax").addClass("ajax");

                    // Tie onclick action to all .ajax links
                    $("a.ajax").click(function() {
                        var link = this;

                        // Save href for future restore
                        var href = $(link).attr("href");

                        // Replace href with '#'
                        $(link).attr("href", "#");

                        // Do actual AJAX
                        $.getJSON(href, function(json) {

                            // Restore href
                            $(link).attr("href", href);

                            // Check JSON response
                            if (json.answer) {
                                $("#answer").text(json.answer);
                            }
                            else {
                                $("#answer").text("Error");
                            }
                        })
                    });
                });
            </script>
            <style>
                a.ajax {
                    text-decoration: none;
                    border-bottom: 1px dashed;
                }
                a.ajax:hover {
                    text-decoration: none;
                    border-bottom: 1px dashed;
                }
            </style>
        </head>
        <body>
            <a href="/ajax" class="no-ajax">Say Hello</a>
            <div id="answer"><%= stash('answer') %></div>
        </body>
    </html>
