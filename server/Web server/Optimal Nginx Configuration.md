Optimal Nginx Configuration

In a standard configuration, Nginx can run at very high loads. Nevertheless, the efficiency of its operation can be greatly improved by tuning its parameters. This setting is called tuning (tuning, adjustment).

How to tune and tune Nginx

Usually the configuration file is called nginx.conf . You can find it:

Debian

/ etc / nginx / nginx.conf
Freebsd

/ usr / local / etc / nginx / nginx.conf
The settings file usually looks like this:

user www-data;
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    ...
}
Optimization of parameters

Processing of connections

The maximum number of connections that Nginx can maintain simultaneously are determined by the product of the two parameters:

Total connections = worker_processes x worker_connections
Nginx worker_processes and worker_connections
worker_processes auto;
# Specifies the number of workflows. It's better to install it in auto in newer versions.

worker_connections 1024;
# Sets the maximum number of connections per workflow. The values ​​from 1024 to 4096 should be selected.

The use directive sets the method for selecting connections. For different operating systems, you need to use different methods.

Linux

use epoll
Freebsd

use kqueue
Nginx epoll
By default, Nginx will try to select the most efficient method by itself.

Query Processing

multi_accept on;
# Will accept the maximum number of connections possible

sendfile on;
# The sendfile method is more efficient than the standard read + write method

tcp_nodelay on;
tcp_nopush on;
# Will send the headers and the beginning of the file in one package

File information

Nginx can cache information about the files with which it has to work (for example, css styles or pictures). If there are a lot of calls to such files, caching can greatly speed up this process.

open_file_cache max=200000 inactive=20s;
# Specifies the maximum number of files that information will be contained in the cache

open_file_cache_valid 30s;
# Determines after what time the information will be removed from the cache

open_file_cache_min_uses 2;
# Will cache information about those files that were used at least 2 times

open_file_cache_errors on;
# Will cache information about missing files

Logging

The main log is better to disable to save disk operations, and the error log is better to switch to logging only critical situations.

access_log off;
error_log /var/log/nginx/error.log crit;
Compression Gzip

It is necessary to use compression, it will significantly reduce traffic. To check if compression is enabled, you can use Gzip checker .

gzip on;
gzip_disable "msie6";
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;
# Will compress all files with the listed types

Customer Processing

Keepalive connections avoid the need to re-create a connection between the client and the server.

keepalive_timeout 30;
# Wait for 30 seconds before closing the keepalive connection

keepalive_requests 100;
# Maximum number of keepalive requests from one client

Many problems can create slow (tupyaschie) customers. Slow transfer of the request body from the client to the server and the unexpected closing of connections by the client can create a large number of unnecessary connections on the server.

reset_timedout_connection on;
# If the client stops responding, Nginx will discard the connection with it

client_body_timeout 10;
# Wait for 10 seconds for the request body from the client, and then drop the connection

send_timeout 2;
# If the client stops reading the response, Nginx will wait 2 seconds and discard the connection

Limit the sending of large requests to the server (for example, downloading large files), if this is not provided by the site.

 client_max_body_size  1m;
# In this case, the server will not accept requests larger than 1MB

After editing the settings, you must reboot:

nginx -s reload
The whole configuration

worker_processes  auto;
events {
    use epoll;
    worker_connections 1024;
    multi_accept on;
}
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log off;
    error_log /var/log/nginx/error.log crit;

    keepalive_timeout  30;
    keepalive_requests 100;

    client_max_body_size  1m;
    client_body_timeout 10;
    reset_timedout_connection on;
    send_timeout 2;
    sendfile on;
    tcp_nodelay on;
    tcp_nopush on;

    gzip on;
    gzip_disable "msie6";
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;

    open_file_cache max=200000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
The most important

The biggest effect on visitors is the inclusion of gzip compression .