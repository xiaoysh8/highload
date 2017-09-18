Balancing backends with Nginx

Load balancing between multiple applications, backends and servers is part of the process of optimizing resources , improving performance and resiliency of the service. Nginx load balancing

Nginx as load balancer

This web server is considered one of the most popular and productive solutions, because it has the widest functionality and flexibility when setting up . So Nginx is often used for load balancing.

There are several approaches and implementations, but first check the availability of the ngx_http_upstream_module :

nginx -v
# All installed and excluded modules will be contained in the output

If it is not present, you will need to rebuild Nginx by adding this module.

After that, you can start configuring the web server. To enable balancing, add the upstream directive ( http section ) to the Nginx configuration file:

upstream backend  {
  server backend1.somesite.com;
  server backend2.somesite.com;
  server backend3.somesite.com;
}
# You can assign multiple upstream directives

Now you need to specify the redirection of the required group:

server {
  location / {
    proxy_pass  http://backend;
  }
}
# Redirection can also be specified in the directives fastcgi_pass, memcached_pass, uwsgi_pass, scgi_pass

In addition, Nginx supports additional parameters and methods for load balancing.

Selecting the Balancing Method

Nginx offers several methods for load balancing. Nginx least_conn

Round-robin

The default web server distributes requests evenly between the backends (but with the weights). This is the standard method in Nginx, so the inclusion directive is missing.

least_conn

Requests are first sent to the backend with the fewest active connections (but taking into account the weights):

upstream backend {
    least_conn;

    server backend1.somesite.com;
    server backend2.somesite.com;
}
# If the number of active connections is the same, then the round-robin

Hash and IP hash

With this method, you can create a kind of permanent connection between clients and backends. Nginx hash

For each query, Nginx computes a hash that consists of text, web server variables, or a combination of these, and then maps it to the backend:

upstream backend {
   hash $scheme$request_uri;

   server backend1.somesite.com;
   server backend2.somesite.com;
   server backend3.somesite.com;
}
# To calculate the hash, use the scheme (http or https) and the full URL

IP hash works only with HTTP, it is a predefined version in which the hash is computed by the client's IP address:

upstream backend {
   ip_hash;

   server backend1.somesite.com;
   server backend2.somesite.com;
   server backend3.somesite.com;
}
# If the IPv4 client address, then the first three octets are used for the hash, if IPv6, then the entire address

Weight backends

Nginx backends weight
If the stack has certain backends stronger than others, weights will be useful:

upstream backend {
    server backend1.somesite.com weight=10;
    server backend2.somesite.com weight=5;
    server backend3.somesite.com;
    server 192.0.0.1 backup;
}
# By default weights are equal to 1

In this example, out of every 16 requests, the first backend will process 10, the second 5, and the third one. In this case, the backup server will receive requests only if the three main backends are unavailable.

Monitoring

If Nginx considers that the backend server is unavailable, it temporarily stops sending requests to it. For this, there are two directives:

max_fails - specifies the number of unsuccessful connection attempts after which the backend is considered inaccessible;
fail_timeout - the time that the server is considered inaccessible.
The parameters look like this:

upstream backend {                
    server backend1.somesite.com;
    server backend2.somesite.com max_fails=3 fail_timeout=30s;
    server backend3.somesite.com max_fails=2;
}
# Default settings: 1 attempt, 10 seconds timeout

The most important thing

Choosing the appropriate balancing method will allow you to more evenly distribute the load. Do not forget about the scales of backends, monitoring and fault tolerance of servers.