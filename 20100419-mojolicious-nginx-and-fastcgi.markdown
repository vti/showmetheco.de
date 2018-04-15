Title: Mojolicious, Nginx and FastCGI
Tags: mojolicious, nginx, fastcgi, fcgi
Comments: no

To use [Mojolicious](https://metacpan.org/pod/Mojolicious) under nginx you should first install [Mojo::Server::FCGI](https://metacpan.org/pod/Mojo::Server::FCGI). We are going to use fcgi\_prefork server, since we need a manager. Nginx configuration file needs a couple tweaks. Here is a config:

    server {
        listen   80; 

        server_name  example.com;

        access_log  /var/log/nginx/example.com/access.log;

        location / { 
            root /var/www/nginx/example.com;

            # Our Mojolicious app runs on this address
            fastcgi_pass   127.0.0.1:3000;

            # Default fastcgi parameters
            include /etc/nginx/fastcgi_params;

            # Fixes
            fastcgi_param  PATH_INFO          $fastcgi_script_name;
        }   
    }

Then run your [Mojolicious](https://metacpan.org/pod/Mojolicious) application as:

    $ perl my_app fcgi_prefork
    Server available at :3000.

Start nginx. Enjoy.
