##Nginx缓存
Nginx可以缓存请求，它这个功能比Vasrnish实现起来要简单的多。

什么应该被缓存？

基于Server的缓存，指的是相同页面的每次请求，不必要重新生成，如Wordpress的文章列表，读取时，也许每次都要花点时间来处理。而缓存，否不必，应用程序生成页面，并把结果存放在内存里，下次用户访问同一个页面，就直接返回缓存的版本。经过指定的一段时间，这一内容会从缓存中删除，并由应用重新生成。

 Nginx cache

几乎可以为所有未登录的用户缓存所有web页面，最佳运用场合是网站的公共内容。

启用Nginx缓存

首先，要设置最大缓存值（最大缓存上限），这个在nginx.conf文件的http部分进行配置。

	http {
	...
	proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=all:<b>32m</b>;
	...
	}
	# 缓存设为32M,存放在/var/cache/nginx目录下

记住要创建缓存目录

mkdir /var/cache/nginx


Hosts设置

要缓存正常生效，需要新建一个缓存主机host监听端口80，然后把主服务器设置到其他端口，如81，缓存服务器接受请求，并返回数据。

	Cache host
	
	server {
	        listen 80;
	
	        location / {
	                proxy_pass http://127.0.0.1:81/;
	                proxy_cache all;
	                proxy_cache_valid any 1h;
	        }
	}
	# 页面将存放在缓存中1小时
	
	Main host
	
	server {
	        listen 81;
	
	        location / {
	                # fpm and other
	        }
	}
	# 应用配置，监听端口为81
	
###Cookies和personalization


Many websites have pages with personalized content blocks. SSI technology allows to implement advanced caching for cases when large numbers of personalized content blocks are used. In the simplest case, cache can be disabled if user has cookies.

很多网站，登录用户各有各的内容，当网站有很多个人化的内容区块时，SSI技术可以实现更高级的缓存。最简单的例子，如果用户有cookies信息，那就禁用缓存。

	server {
	        listen 80;
	
	        location / {
	                if ($http_cookie ~* ".+" ) {
	                        set $do_not_cache 1;
	                }
	                proxy_cache_bypass $do_not_cache;
	                proxy_pass http://127.0.0.1:81/;
	                proxy_cache all;
	                proxy_cache_valid any 1h;
	        }
	}


出错信息

在一段时间内缓存失败的请求信息是比较好的作法，免得服务器不断的去查找不存在的资源或内容。

	
	server {
	        listen 80;
	
	        location / {
	                if ($http_cookie ~* ".+" ) {
	                        set $do_not_cache 1;
	                }
	                proxy_cache_bypass $do_not_cache;
	                proxy_pass http://127.0.0.1:81/;
	                proxy_cache all;
	                proxy_cache_valid 404 502 503 1m;
	                proxy_cache_valid any 1h;
	        }
	}

fastcgi缓存

nginx可以缓存fastcgi返回的内容，启用这一功能，需要配置对应的参数(nginx.conf的http部分)


fastcgi_cache_path /var/cache/fpm levels=1:2 keys_zone=fcgi:100m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";
# 设置最大缓存上限为100MB

别忘了创建文件夹

mkdir /var/cache/fpm

增加规则到主服务器配置中：

	server {
	    listen   80;
	
	    location ~ \.php$ {
	        fastcgi_pass unix:/var/run/php5-fpm.sock;
	        fastcgi_index index.php;
	        include fastcgi_params;
	        fastcgi_cache fcgi;
	        fastcgi_cache_valid 200 60m;
	    }
	}
# 响应为200的请求，将缓存60分钟

结论：充分利用好缓存，配置很简单，在系统性能上有10倍的增效。
