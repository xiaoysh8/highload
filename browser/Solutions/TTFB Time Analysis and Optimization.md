TTFB Time Analysis and Optimization

Broadly speaking, TTFB is a metric that shows the time until the first byte (network packet) of the web page is received after the request is sent from the client. TTFB

The dimension includes a DNS query, the time it takes to connect to the server, and the time it waits for the processed request (processing, repacking, sending the page). Interestingly, the term is often confused with the response time of the server - this indicator makes it possible to evaluate the response rate to the HTTP request in the absence of a network delay. TTFB is affected by almost everything: network problems and delays, incoming traffic volume, web server settings, volume and content optimization ( graphics quality , css / js / html size ).

Obviously, not all the above points are easy to influence. You are unlikely to be able to independently improve the quality of the network, and high traffic to the website can be something bad only in the case of DDoS-attacks. The only thing that is really affected is the backend, so let's deal with tuning Nginx.

TTFB Analysis

In the vastness of the World Wide Web, a large number of resources are devoted to checking the speed of loading web pages. One of the most popular and respected is WebPageTest.org .

It provides more than exhaustive information about connection time, TTFB, TLS / SSL initialization time (if used), and about loading individual web page elements.

To check the download speed and a whole stack of additional parameters, you can use the browser. And in Chrome, and in FireFox, and even in Safari there are corresponding developer tools.

Optimizing Nginx

The optimal settings for Nginx are presented in this article. Once again, briefly go through the already known parameters and add a few new ones that directly affect the TTFB.

Connections

First, you need to specify the number of "employees" Nginx. worker_processes Nginx

Each workflow is capable of processing multiple connections and is attached to the physical cores of the processor. If you know exactly how many cores in your server, you can set their number yourself, or trust Nginx:

worker_processes auto;
# Determine the number of work processes

In addition, you must specify the number of connections:

worker_connections 1024;
# Determine the number of connections per workflow, ranging from 1024 to 4096

Inquiries

To enable the web server to handle the maximum number of requests, you must use the default multi_accept directive :

multi_accept on;
# Workflows will accept all connections

It is noteworthy that the function will be useful only if a large number of requests simultaneously. If there are not so many requests, it makes sense to optimize workflows so that they do not work idly:

accept_mutex on;
# Workflows will receive connections in turn

Improving the TTFB and the response time of the server directly depends on the tcp_nodelay and tcp_nopush directives :

tcp_nodelay on;
tcp_nopush on;
# Activation of the tcp_nodelay and tcp_nopush directives

If you do not go into too much detail, both features allow you to disable some TCP features that were relevant in the 90s, when the Internet was just gaining momentum, but do not make sense in today's conditions. The first directive sends the data as soon as it is available (bypasses the Neigle algorithm). And the second makes it possible to send the response header (web page) and the beginning of the file, waiting for the packet to be filled (i.e., includes tcp_cork ). So the browser will be able to start displaying the web page earlier.

At first glance, the functions contradict each other. Therefore, the tcp_nopush directive should be used in conjunction with sendfile . In this case, the packets will be filled before sending. the directive works much faster and more optimally than the read + write method . After the package is full, Nginx automatically turns off tcp_nopush , and tcp_nodelay causes the socket to send data. Enabling sendfile is very simple:

sendfile on;
# Enabling a more efficient, as compared to read + write, method of sending files

So the combination of all three directives reduces the load on the network and speeds up the sending of files.

Buffers

Another important optimization affects the size of the buffers - if they are too small, Nginx will often access the disks, too large - RAM will quickly fill up. Nginx Buffers

To do this, you need to configure four directives. client_body_buffer_size and client_header_buffer_size specify the size of the buffer to read the body and the request header of the client, respectively. client_max_body_size specifies the maximum size of the client request, and large_client_header_buffers specifies the maximum number and size of the buffers to read the large request headers.

The optimal buffer parameters will look like this:

client_body_buffer_size 10K;
client_header_buffer_size 1k;
client_max_body_size 8m;
large_client_header_buffers 2 1k;
# Buffer size 10KB per request body, 1KB per header, 8MB per request and 2 buffers to read large headers

Timeouts and keepalive

Correct setting of time-out and keepalive can also significantly improve server responsiveness.

The directives client_body_timeout and client_header_timeout specify the wait time for reading the body and the request header:

client_body_timeout 10;
client_header_timeout 10;
# Set the timeout, in seconds

In this case, if there is no response from the client using reset_timedout_connection, you can specify Nginx to disable such connections:

reset_timedout_connection on;
# Disabling connections that have exceeded the timeout

The keepalive_timeout directive specifies the timeout before disconnecting, and keepalive_requests limits the number of keepalive requests from one client:

keepalive_timeout 30;
keepalive_requests 100;
# Set the waiting time to 30 seconds and limit 100 requests to the client

Well, send_timeout specifies the waiting time for sending a response between two write operations:

send_timeout 2;
# Nginx will wait for an answer of 2 seconds

Caching

Enabling caching will greatly improve the response time of the server. Nginx cache

Methods are described in more detail in the material on caching with Nginx, but in this case, the inclusion of cache-control is relevant . Nginx is able to send a request for caching of rarely-changed data, which is often used, on the client side. To do this, add the following line to the server section:

location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {expires 365d;}
# Specify the file formats and the length of the cache storage

Also, you can cache information about frequently used files:

open_file_cache max=10000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;
# Allows caching descriptors of 10,000 files within 30 seconds

open_file_cache specifies the maximum number of files to be stored and the retention time. open_file_cache_valid specifies the time after which it is necessary to check the relevance of the information, open_file_cache_min_uses specifies the minimum number of requests to the file from the client side, and open_file_cache_errors includes caching of file search errors.

Logging

This is another feature that can slightly reduce the performance of the entire server and, accordingly, the response time and TTFB. So the best solution is to disable the main log, and save the information only about critical errors:

access_log off;
error_log /var/log/nginx/error.log crit;
# Disable basic logging

Compression Gzip

The usefulness of Gzip is difficult to exaggerate. Compression significantly reduces traffic and offloads the channel. But it also has the reverse side - it takes time to compress. So to improve the TTFB and the response time of the server it will have to be disabled. Gzip

At this stage, we can not recommend disabling Gzip, since compression improves Time To Last Byte, that is, the time it takes to fully load the page. And this is in most cases a more important parameter. The improvement in TTFB and server response time will be significantly affected by the large-scale implementation of HTTP / 2 , which includes built-in methods for header compression and multiplexing. So in the future, maybe turning off Gzip will not be as noticeable as it is now.

Optimizing PHP: FastCGI in Nginx

All modern sites use server technologies. PHP, for example, which is also important to optimize . Typically, PHP opens a file, checks and compiles the code, then executes it. There can be a lot of such files and processes, so PHP can cache the result for rarely edited files with the OPcache module. And Nginx, connected to PHP using the FastCGI module, can store the result of executing the PHP script for instant sending to the user.

The most important

Optimizing resources and correct web server settings are the main factors affecting TTFB and server response time. Also, do not forget about regular software updates to stable versions that carry optimization and performance improvements.