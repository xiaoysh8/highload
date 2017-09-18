nginx 最佳配置
=============

>nginx默认配置，已经可以应付非常高的负载了。不过，通过调整其配置，仍可极大的提高nginx的性能。

怎样去配置调整Nginx?
-------------------

通常情况下，nginx的配置文件叫nginx.conf。可以在以下位置找到它:

Debian

/etc/nginx/nginx.conf

Freebsd

/usr/local/etc/nginx/nginx.conf

配置文件看起来象这样:

~~~

	user www-data;
	worker_processes 1;
	
	events {
	    worker_connections 1024;
	}
	
	http {
	    ...
	}

~~~

参数
--------
####连接
\#Nginx同时能保持的最大连接数，取决于两个参数

	Total connections = worker_processes * worker_connections

![](http://ruhighload.com/nginx.jpg)

	worker_processes auto;

\#指定同时保持多少处理线程数，最好设为"auto"。

	worker_connections 1024;

\#指定单个线程可以保持的最大连接数。可以选择从1024到4096。

\#指令use设置连接池的方式，不同操作系统，应该选择不同的方式。

	Linux
	
	use epoll
	
	Freebsd
	
	use kqueue

![](http://ruhighload.com/nginx.epoll.jpg)

默认情况下，Nginx会自动选择合适的方式。

####请求处理

	multi_accept on;

\# 自动接受最大数量的可连接数

	sendfile on;

\# sendfile方式比标准的read+write方式更有效。

	tcp_nodelay on;
	tcp_nopush on;

\# 会将文件的开始信息和发送头(header)放在一个包里发送。

####文件信息

Nginx可以缓存css,图片文件,如果重复请求,则可极大提高相应速度。

	open_file_cache max=200000 inactive=20s;

\# 指定最大的缓存文件数量

	open_file_cache_valid 30s;

\# 指定缓存失效时间

	open_file_cache_min_uses 2;

\# 将缓存被请求过2次的文件

	open_file_cache_errors on;

\# 将缓存找不到的文件信息

####日志

主日志关闭省掉磁盘读写，错误打开，改为critical模式。

	access_log off;
	error_log /var/log/nginx/error.log crit;

####Gzip 压缩

一定要使用压缩，它会显著的降低传输的数据量，检查是否已启用，可用Gzip checker。

	gzip on;
	gzip_disable "msie6";
	gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;

\# 会压缩所有上面列出的文件类型。

####Processing customers

Keepalive connections allow you to avoid having to re-create the connection between the client and the server.

	keepalive_timeout 30;

\# Will wait 30 seconds before closing the keepalive connections

	keepalive_requests 100;

\# The maximum number of keepalive requests from the same client

Many problems can create slow (tupyaschie) customers. Slow transfer request body from client to server and unexpected closing of client connections can create a large number of extra connections on the server.

	reset_timedout_connection on;

\# If the client is no longer responding to read, Nginx will reset the connection with him

	client_body_timeout 10;

\# Will wait for 10 seconds, the request body of the client, and then reset the connection

	send_timeout 2;
\# If the client stops reading the answer, Nginx will wait 2 seconds and reset the connection

Limit sending large queries to the server (for example, downloading a large file), if it is not provided by the site.

	client_max_body_size  1m;

\# In this case, the server will not take more than 1MB in size requests
After configuration changes need to reboot:

	nginx -s reload

####All configuration

~~~
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
~~~

The most important

The biggest effect on visitors will switch gzip compression .