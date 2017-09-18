Distributed cache based on Nginx and proxy_cache

When using caching, the disk space will end sooner or later. In this case, cache cleaning is usually used. For example, delete all files that were requested more than 7 days. In Nginx, this is configured like this:

proxy_cache_path /var/cache/nginx inactive=7d levels=1:2:2 keys_zone=local_cache:50m max_size=10g;
# rule for deleting files from the cache

But clearly, this worsen the experience users.

Distributed cache

Alternatively, you can use a distributed cache. In this case, only part of the total cache will be stored on each server. So, the total number of files that can be stored in the cache will be much larger.

This option will be useful when you are giving out a lot of files. Especially large, like video or photos. To apply it costs, for example, within the system of optimization of expenses for CDN .

Query distribution

All you need to do is force nginx to send part of the requests to the neighboring server. First of all, we will collect a simple configuration of the return of files from different servers (without cache).

Let's have two servers:

165.227.150.69
207.154.211.168
If the request came to the first server, we want to give only half of the requests from it. And send the second half to the second server. The same is true for the second server, only for the second half of the queries: Distributed cache based on Nginx

The distribution is done on the basis of upstream hash :

upstream cluster {
    hash $request_uri consistent;

    server 165.227.150.69:81;
    server 207.154.211.168:81;
}

server {
	listen 80;
	location / {
		proxy_pass http://cluster;
	}
}

server {
	listen 81;
	root /var/www/html;

	rewrite ^(.*)$ index.php;

        location ~* \.(php)$ {
            fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME /var/www/html/index.php;
        }
}
# Query distribution in Nginx based on a consistent hash

Here we are:

Using the block upstream clusterdetermine the list of servers. The original request will be sent to them. So, one of them - this is the server itself, and the second is its neighbor.
hash $ request_uridetermines the principle of the distribution of requests (in our case - consistent hashing ).
The first virtual server listens on port 80 . It simply sends a request to upstream .
The second virtual server serves the request (in the example - php script).
Adding Cache

Now it's enough to add an intermediate virtual server in our configuration, which will cache the data. We will transfer the virtual server from port 81 to 82 . And port 81 we 'll take server for caching:

upstream cache_cluster {
    hash $request_uri consistent;

    server 165.227.150.69:81;
    server 207.154.211.168:81;
}

server {
	listen 80;
	location / {
		proxy_pass http://cache_cluster;
	}
}


proxy_cache_path /var/cache/nginx levels=1:2:2 keys_zone=local_cache:50m max_size=10g;
proxy_cache_key "$uri";
server {
	listen 81;

        location / {
            proxy_cache local_cache;
            proxy_pass http://127.0.0.1:82;
	    proxy_cache_valid 200 21d;
	    proxy_cache_methods GET;
        }
}

server {
	listen 82;
	root /var/www/html;

	rewrite ^(.*)$ index.php;

        location ~* \.(php)$ {
            fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME /var/www/html/index.php;
        }
}
# Distributed query cache

Now, apart from the distribution, requests are also cached. Part of the cache files will be added to one server. The second part - on the other. The storage space for cache was twice as large. What we wanted to get.

Internal traffic

Note.One server proxy some requests from the second one. This can cause problems if the server is geographically remote. In this case, it makes sense to use server groups that are located side by side (in one DC).

Traffic between nodes will grow significantly if there are a lot of servers. Also it is solved by grouping of servers (for example, on 10 in one group).

Availability

Now the servers depend on each other. And if one breaks, this oneusers will see the error right away. To ensure high availability, you should always put (at least) a pair of servers that will act as copies of each other. Then, if one fails, you can automatically switch to another one.

TL; DR

Nginx allows you to configure a distributed cache between multiple servers. It should be used to increase the efficiency of the cache (more space = more hitrate).
You should not use more than 10 servers in one distributed cache group.
To ensure high availability, you should have a working copy of each server.