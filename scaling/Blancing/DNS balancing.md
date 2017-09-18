##DNS负载均衡

浏览器发出请求后，DNS服务器是第一个收到请求的节点，它返回应用的实际ip地址。然后浏览器再根据这个地址请求应用服务并获得响应。


IP地址存在一个A域名记录里，如
ruhighload.com 在 A 188.226.228.90

通常指定了一个A记录，DNS总是返回相同的IP地址。不过，也可以改变给定的IP地址，访客可以从同一个域名获得不同的IP地址。

###Round robin


      DNS可以允许在A记录中，指定多个IP地址：
      ruhighload.com IN A 188.226.228.90
      ruhighload.com IN A x.x.x.x
      ruhighload.com IN A y.y.y.y
      # xxxx and yyyy - 其他前端的IP地址

在这个例子里，不同的客户端，访问域名时，会返回不同的IP地址，将负载分配到不同的应用服务器上。Round Robin是按轮流的方式来分配地址，不过这个没有一个统一算法，所以不能完全指望它。

IP地址缓存在本地的dns里，如果服务器中的一台down机了，那要等很长一段时间缓存更新，才能恢复。所以Round Robin应该和容灾系统一起使用。

在A记录中，设置较小的ttl值，以便IP地址快速更新。优化值是几分钟。

###Geo DNS
DNS服务器可以根据用户的地理位置，返回对应较近的ip地址。这对于网站用户在地理分布上很分散的情况，是很有用的。

假如你很多访客来自美国和欧洲，而只有欧洲有服务器，那么在美国做一个服务器镜相就很有用。这一功能用于CDN。

###Bind GeoDNS

作为一种比较流行的绑定服务器的方法，可以使用[http://www.caraytech.com/geodns/](http://www.caraytech.com/geodns/ "geodns")，它允许指定不同的地理数据库配置。地理数据库可以用[https://www.maxmind.com/en/geoip2-country-database](https://www.maxmind.com/en/geoip2-country-database "Max yet Mind")

	...
	view "usa" {
	      match-clients { country_US; country_CA; country_MX; };
	      recursion no;
	      zone "ruhighload.com" {
	            type master;
	            file "pri/ruhighload.usa.db";
	      };
	};
	
	view "other" {
	      match-clients { any; };
	      recursion no;
	      zone "ruhighload.com" {
	            type master;
	            file "pri/ruhighload.other.db";
	      };
	};
	# 域名使用GeoDNS
 
结论：使用dns轮询来为多个服务器进行负载均衡，为ttl配置一个较短的时间，再看一看前端负载均衡的方案。

