Varnish is a flexible web server caching solution.

Installation and launching

Varnish can be installed on all known Linux and Unix distros. For Debian, it looks like this:

apt-get install varnish
General architecture

Varnish + Nginx
Varnish usually listens 80 port. This means it will receive all requests from clients. If it's undesirable for Varnish to do so (for example, it doesn't support SSL), you can choose a different port (6081) and setup a chain like Nginx[80]-Varnish[6081]-Nginx[81].

Enabling cache

If you're using Debian, Varnish startup parameters should be configured in a file:

nano /etc/default/varnish
Parameter DAEMON_OPTS should contain:

DAEMON_OPTS="-a :80 \
-T localhost:6082 \
-f /etc/varnish/default.vcl \
-S /etc/varnish/secret \
-t 120 \
-s malloc,256m"
# Thus, setting default cache size to 256MB and default cache time to 120 seconds

VCL

Varnish VCL
VCL provides many options to tune cache operation. Configuration file is typically located here:

nano /etc/varnish/default.vcl
File is divided into several sections, but to enable VCL, the first thing to do is to setup backends. Backend is a server address, on which Varnish will send requests if cache doesn't contain the required data. Backend responses will be cached.

backend www {
  .host = "127.0.0.1"; 
  .port = "81";
}
# Varnish will send requests to port 81

Now, we should make our main website listen to port 81. If you're using Nginx:

server {
        listen 81;
        ...
}
Restart Varnish:

/etc/init.d/varnish restart
VCL is practically a complete development language. All instructions should be written in default.vcl file (remember to uncomment the relevant section).

Cookies and static content

By default Varnish will not cache requests with cookies enabled. But it ruins the thing for static files. Thus, you can remove cookies in vcl_recv subroutine:

sub vcl_recv {
...
        if (req.url ~ "\.(css|js|png|gif|jp(e)?g)") {
                unset req.http.cookie;
        }
...
return (lookup);
}
Large files

If you do not want to choke cache with huge files (for example, your website has video/music), 'pass' instruction should be used:

sub vcl_recv {
...
        if (req.url ~ "\.(mp4|mp3|avi)") {
                return (pass);
        }
...
return (lookup);
}
Cache time (ttl))

In VCL configuration you can override default cache time. It means that different time periods can be used for certain conditions:

sub vcl_recv {
  set beresp.ttl = 2m;
        if (req.url ~ "\.(png|jpe?g|gif)") {
                set beresp.ttl = 1h;
        }
}

sub vcl_fetch {
   set beresp.ttl = 1h;
}
# Everything is cached for 2 minutes, images — for 1 hour

Dynamic pages

Varnish supports a special caching technique for dynamic pages with the use of ESI.

The most important

Using cache is the best way to reduce server load in tens or even hundreds of times.