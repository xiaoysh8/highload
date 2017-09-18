高负载架构
===========

对高负载的应用，架构是基础。理解应用架构，就决定的项目95%的成功。这也意味着应用承载压力的能力。

###设计原则 Design principles

创建一个大型成功的web应用，你必须理解大型系统的构建原则。

#####动态Dynamics

你永远不知道明天应用的用户会有多少，也许你的用户数量会增加5倍。也许原来不起眼的程序迅速受到欢迎，这会成为一个新问题。系统变的越大，长远规划就显得效果甚微。

一个大项目的成功并不意味着计划所有的细节，主要的努力方向应该保证系统的灵活性。这是任何快速成长系统的最重要的特点。

#####逐渐成长Gradual growth

不会企图预测几年后的用户数量，这点也同样适用于应用的架构。 成功的开发基础就是渐进的解决方案，这同样指软件及硬件。

如果你刚开始一个新应用，提供一个可以支持百万访问者的架构是没有意义的。用云服务器创建新项目，可以降低成本，简化服务器管理。

很多云服务器支持私有网络，这允许在一个云里使用多个服务器。这样，就可以在云服务器中，实现第一步扩展。

#####使解决方案更简单Simple solutions

简化解决方案要是极其困难的，不过，更好的作法是去花时间努力简化开发和用户使用流程，灵活的系统不会太复杂。

#####逐渐改变Progressive changes

开发一个大型项目象加载一个渐进的JPEG图像。整个改进过程并不是那么规律。你不得不持续的重新考虑不同解决方案，从这种方法，切换到另一种。

#####95th percentile

![](https://i.onthe.io/shpzkl7ilgu5jiebg.c7dc113c.jpg)

使用95%法则。把时间花在95%的问题上，而余下的5%通常对系统来说是极个别用户的问题。如:

- 某些用户在社交平台上有上百万的链接，如果这样的用户不超过5%，那就依据一定的条件，不到这个杠杠就不去解决这个问题。
- 有少于5%的用户，上传非标格式的视频文件，显示错误信息，而不是浪费时间去解决这个问题。
- 有少于5%的用户浏览器禁用了cookies,js，对这些用户关闭网站，显示相应操作或更新浏览器，而不是调整整个网站。

关注重点。记住，总有要解决的问题，不必把时间花在小概率事件上纠结。确定合适的优先序列，去解决那些绝大数用户遇到的问题。


#####架构解决方案Architectural solutions

扩展任何web应用是一个渐进的过程：

#####负载分析

判断最多被请求服务的部分作为主要负载。
加强并优化相应的结点。
重复第一章节。

- 简化应用架构

    应用开始通常跑在一个服务器上，web服务，数据库和应用本身在同一台机器上。

![](https://i.onthe.io/shpzkl76lkr6i9q1t.67c398ad.jpg)

这种方式很合理，节约了时间和成本。如果你怕承受不住一开始的负载，就租一个配置高的服务器。在一些特殊情况下，如果你很确定有一个极大的负载，那直接按以下方式来做：


- 分离数据库

数据库通常是先受到高负载考验的结节，每一个从用户的访问要创建10至100个数据库查询。

将数据库传送到另一台服务器上，可以提高查询效率，减小查询对其他部分性能的影响（PHP,Nginx等等)。使用ip可以直接连接另一台数据库:

	mysql_connect('10.10.0.2', 'user','pwd');
	# 10.10.0.2 是内网 MySql IP 地址 

	- 转移数据工作中需要暂停服务

	数据库迁移到另一台服务器上需要一定时间，这会影响到正在运行的服务。
	
用一个简单方法，在网站上提示要作维护，在某个时间断暂停了服务，进行数据迁移。一般在晚上用户较少时进行比较好。使用复制同步机制转移数据库数据。此时，master就是原来的服务器，新的服务器就是slave。在设置完成后，改变应用查询的地址即可。下面介绍怎么样在不断数据服务的情况下进行mysql主从复制。在主从复制完成之后，确保最优配置参数，以达到最大性能。

数据库扩展在项目开发中是一个极大的挑战。有许多工作要做：数据库反范式，主从配置，分表及其他。这里是关于web应用的扩展方法。


- 分离web服务器
>The web server is next in line. Its detachment to a separate node leaves more resources for an application (PHP in this example): Web server detachment

In this case you have to configure the application deployment on the Nginx server and the PGP server. PHP will be a backend server. Nginx will serve static files alone, and PHP server will be busy processing scripts only. Nginx allows to connect to the backend with IP address:

server {
  server_name thehighload.com;

  root /var/www/thehighload;
  index index.php;

  location ~* \.(php)$ {
        fastcgi_pass 10.10.10.1:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
# Nginx will serve all static files without requests to the backend

If you are using a file upload, you will need to detach file storage into a single node (about this read below).

4. Several PHP backends

When the load increases, web application begins gradually to work slower. At some point, application implementation will be the reason itself. Then it is necessary to set a couple of PHP backends: Several PHP backends All backends preferred to have the same configuration. Nginx is able to balance the load between them. To do this, you need to create the upstream backend list and use it in the configuration:

upstream backend {
    server 10.10.10.1;
    server 10.10.10.2;
    server 10.10.10.3;
}

server {
  server_name thehighload.com;

  root /var/www/thehighload;
  index index.php;

  location ~* \.(php)$ {
        fastcgi_pass backend;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
# Nginx will evenly distribute the load between the specified backends

You can also use weights of the backends if some of them are more powerful than others:

upstream backend {
    server 10.10.10.1 weight=10;
    server 10.10.10.2 weight=2;
    server 10.10.10.3 weight=4;
}
# Out of every 16 requests the first backend processes 10, the second 2, and the third 4

Sessions

Once you start using multiple backends, requests from one user would be transferred to different servers. This will require the usage of a single repository for sessions, for example, Memcache.

5. Caching

Connection caching servers is one of the most simple tasks: Architecture caching

Memcache allows multiple servers to use the standard library:

<?
$m = new Memcache;
$m->addServer('10.5.0.1');
$m->addServer('10.5.0.2');
...
$m->get('user1')
# Connecting memcache to multiple servers at once

Memcache independently distributes the load between the servers. Is uses consistent caching for the purpose. You will need to monitor the evictions and add new equipment in time.

6. Task queues

Task queues let you perform difficult operations asynchronously without slowing the main application. Architecturally, it looks like this: Task queues architecture

The queue server receives tasks from the application. Worker servers process all tasks. Their number should be increased when the average number of jobs in the queue will gradually grow.

7. DNS Balancing

DNS supports round-robin balancing. This allows you to specify multiple IP addresses to hosting web servers (usually referred to as front end): DNS round robin

To use this mechanism, it is necessary to install several identical front ends. Then A records should be specified in the DNS:


....
thehighload.com    IN  A       188.226.228.90
                  IN  A       188.226.228.9x
                  IN  A       188.226.228.9x
# Using multiple IP addresses for a single A record

In this case, DNS will give different IP addresses to different clients. Thus, the balancing between the front end will occur.

8. File storage

Upload and processing of files usually happen in the backend. When you have multiple backends, it is inconvenient:

It is necessary to remember where the file was uploaded.
Upload and processing files (eg, video or photos) can greatly reduce the performance of the backend.
We'll have to use a server with large disks, in what is usually no need for a backend.
The correct solution is to use separate servers for uploading, storing and processing of files: File storage architecture

In practice, this looks like this:

Providing a separate subdomain for the file server.
You need to deploy Nginx and small application that is able to save (and process, if necessary) files on the server.
Scaling is done by adding new servers and subdomains (images 1, images 2, images 3 etc.).
Files upload

Uploading is convenient when is shifted to the client side. Then the form will send requests to a particular server:

<form action="http://images1.thehighload.com/upload.php" method="post" enctype="multipart/form-data">
  <input type="file" name="file">
  <input type="submit" value="Загрузить">
</form>
Domains can be generated randomly from the existing ones:

<form action="http://images<?=mt_rand(1, 10)?>.ruhighload.com/upload.php" method="post" enctype="multipart/form-data">
...
AJAX uploading

To implement AJAX upload, you will need to set the HTTP headers in applications that receive files:

<?
header('Access-Control-Allow-Origin: http://ruhighload.com');
...
# This will send AJAX requests from thegihgload.com domain to the upload domains

Read further about the structure of storage for photos and media data in large projects.

The most important

Remember — not technology but architecture handles high loads. The technologies itself aren’t that important, but right usage is needed. Gradually scale and read more on architectural solutions.