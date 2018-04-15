Title: Adding ETag caching to your Mojolicious app
Tags: Perl, Mojolicious, ETag, Caching
Comments: no

ETag caching is a kind of resource checksum. When the requested page changes,
checksum changes too. In comparing to the last modified date it is well suited for
dynamic content. A browser caches the page and on the next request sends the
ETag from the previous response. Server checks if the page was changed
and sends the updated page or 304 http status.

In [Mojolicious](https://metacpan.org/pod/Mojolicious) this can be implemented with a simple plugin using
`after_dispatch` hook.

[cut]

    package MyApp::Plugin::HttpCache;
    
    use strict;
    use warnings;
    
    use base 'Mojolicious::Plugin';
    
    use Mojo::ByteStream;
    
    sub register {
        my ($self, $app) = @_; 
    
        $app->hook(
            after_dispatch => sub {
                my $self = shift;
    
                return unless $self->req->method eq 'GET';
    
                my $body = $self->res->body;
                return unless defined $body;
    
                my $our_etag = Mojo::ByteStream->new($body)->md5_sum;
                $self->res->headers->header('ETag' => $our_etag);
    
                my $browser_etag = $self->req->headers->header('If-None-Match');
                return unless $browser_etag && $browser_etag eq $our_etag;
    
                $self->res->code(304);
                $self->res->body('');
            }   
        );  
    }
    
    1;

BTW, ETags could be used not only for caching, but also for user tracking
[http://en.wikipedia.org/wiki/HTTP\_ETag#Tracking\_using\_ETags](http://en.wikipedia.org/wiki/HTTP_ETag#Tracking_using_ETags).
