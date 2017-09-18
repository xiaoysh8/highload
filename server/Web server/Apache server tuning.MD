Apache 服务器优化
================

Apache是最受欢迎的web服务器。通过调节其部分参数，可以极大的提升其性能。

#####配置

Apache配置文件位置：

Debain系统 /etc/apache2/apache2.conf
Freebsd系统 /usr/local/etc/apache22/httpd.conf

#####关闭不用的模块modules

关闭不用的模块，可以在处理http请求时，释放许多系统资源。apache的模块采用的是插件机制，可以通过enabling 和 disabling在配置文件中进行打开或关闭。

一般情况下所需要的模块如下：

	mod_alias
	mod_authz_host
	mod_deflate
	mod_dir
	mod_expires
	mod_headers
	mod_mime
	mod_rewrite
	mod_log_config
	mod_autoindex
	mod_negotiation
	mod_setenvif
	MPM

MPM让你选择处理并发请求的方式。最好选择Worker，多线程MPM。每一个请求，用一个字线程来进行处理。相对于进程，线程对操作系统来说更轻量。所以它可以极大提升服务器性能。

####开启MPM

	nano /etc/sysconfig/httpd
	and uncomment the line:
	
	HTTPD=/usr/sbin/httpd.worker
	Now restart Apache:
	
	/etc/init.d/apache2 restart
	AllowOverride and .htaccess

AllowOverride指令启动.htaccess文件，apache会在每一次响应请求时，查找这个文件。把所有配置放入虚拟主机文件里(Debian系统在/etc/apache2/sites-enabled文件夹中)，如果禁用htaccess:

	AllowOverride None
	MaxClients

####Apache MaxClients

MaxClients指令限制了服务器最多能处理的并发数。这个设置需要按实际情况来调节：一般区间在128到4096之间。

	MaxClients 256

如果服务器接受的请求已达所设置的并发最大值，那新的请求会进入一个队列，这个队列的大小由ListenBacklog指令来设置。

####MinSpareServers, MaxSpareServers and StartServers

These directives allow having processes created in advance, so when requests are received everything is ready to handle them. The settings are valid only for prefork mpm.

StartServers 3
# Apache will create 3 processes on startup

MinSpareServers 3
# Apache will not kill idle processes, if less than three are left

MaxSpareServers 5
# The limit is set to 5 idle processes, the rest will be killed

MaxRequestsPerChild

MaxRequestsPerChild directive limits how many requests can be handled by one child process/thread before it is suspended. By default, this directive has 0 value, which means that created processes/threads will never expire. This parameter allows getting rid of memory leaks' related issues, so it makes sense to enable it:

MaxRequestsPerChild 4096
# After 4096 requests the process is restarted

KeepAlive

KeepAlive requests allow establishing permanent client-server connections. This frees up resources, as connections won't have to be re-established. Be sure to enable this option.

KeepAlive On
KeepAliveTimeout 30
# Enable KeepAlive and set 30-second timeout before closing the connection

In some case user sends only one request. For example — download server. Then KeepAlive may become useless and even harmful, because active KeepAlive makes server wait some time (KeepAliveTimeout) before closing the connection, not doing this immediately.

Compression

Gzip compression
All modern browsers support compression. Enabling gzip will greatly reduce traffic. This should always be done.

AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE text/css
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE application/rss+xml
AddOutputFilterByType DEFLATE application/javascript
AddOutputFilterByType DEFLATE application/x-javascript
Check whether compression is working with Online Gzip checker.

DNS

Apache HostnameLookups
Disable extra DNS queries in Apache:

HostnameLookups Off
# After this change Apache will log client's IP address instead of its host.

Always use IP addresses instead of domain names in Allow From and Deny From directives.

Allow From 1.1.1.1
Deny From 2.2.2.2
The most important

Enabled gzip compression has the biggest effect on the visitors. Often it reduces the amount traffic by up to 70%. Use other parameters to handle more requests.