##SSI缓存动态站点
使用SSI技术进行动态缓存

缓存网站相对静态的内容很容易，不过现在大多网站有大量的个人化内容，如banner,某些区块等。有个SSI技术来解决这样的情况，技术本身很简单，它允许你把页面分成区块，然后只对某些区块进行缓存。

###SSI

假设一个页面，有些静态内容，另加一个banner,banner的内容取决于用户数据，所以这个banner不可以被缓存。所以除了banner内容外，页面的其他部分被缓存了。

SSI只是一个web server内部的特例指令，插入而不是运行请求的结果到其他页面。举例如下：

<!--# include virtual="/authentication.php" -->
\# 页面将插入请求/authentication.php返回的结果，而不是执行这个脚本。

web服务器根据SSI指令，在发送页面到客户端之前，自动替换掉必要的页面区块。可以缓存整个页面和单个区块的内容。当然，这样做没啥意义，为什么一次性能做的事要分成两次？这一功能在缓存机制内部实现，用起来很方便。


Nginx and SSI
Nginx支持SSI很简单：

	server {
	    ...
	    ssi on;
	    ...
	}
	SSI 缓存
	Nginx可以从Memcache中读取数据。
	Nginx allows you to read data from Memcache .
	
	server {
	  location / {
	     set $ memcached_key $ uri; # 指的是查询字符串
	     memcached_pass 127.0.0.1:11211; # 链接到 memcache
	     default_type text / html; # 默认内容类型
	     error_page 404 = @fallback; # 查询失败，无缓存数据
	  }
	
	  location @fallback {
	    proxy_pass backend; 
	  }
	}

只保存必要的数据到Memcache中（nginx本身不能添加数据到Memcache中）

PHP
使用 php,得确保所有数据通过合适的key值存放在Memcache中

	<?
	$memcache = new Memcache();
	ob_start();
	echo "here there must be <b> html </ b>, and it will be a lot";
	$html = ob_get_clean();
	
	$memcache->set($_SERVER['REQUEST_URI'], $html);
	echo $html;
	?>
	\#保存结果到memcache中

上面的例子定义了SSI区块

First, select the necessary blocks in the template and replace them with SSI calls.
第一步，在模板中选取必要的内容区块，并通过SSI调用来替换它们。
	<html>
	<body>
	
	<h1> Test nginx + memcached + ssi </ h1>
	
	<div class = "auth">
	<! - Authorization Block! ->
	<! - # include virtual = "/ authentication.php" ->
	</ div>
	
	<! - The body of a particular page! ->
	<! - # include virtual = "/ somepage.php" ->
	
	</ body>
	</ html>

正如所见，页面上有两个SSI区块，要认证的区块和内容区块。在这个例子里，认证后的区块将显示个人内容，如登录后，显示用户名。内容区块所有人相同。


配置 Nginx

需要配置两个主机：
主服务器由用户直接访问，它通过SSI使用缓存数据。Backend是应用程序所在，它返回php运行后的结果。
     server {
    	listen 8081;

    	location ~* \.(php)$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME /var/www/test$fastcgi_script_name;
        }
    }
    # Backend


    server {
    	listen 80;

    	root /var/www/test;
    	index index.php;

    	location / {
    		# 全部post提交数据发向后端应用程序，不会缓存
    		if ($request_method = POST) {
    			proxy_pass http://127.0.0.1:8081;
    			break;
    	        }

    		# 启用SSI
    		ssi on;
    		default_type text/html;

    		# 启用Memcache
    		set $memcached_key "$uri";
    		memcached_pass localhost:11211;
    		proxy_intercept_errors  on;
    		error_page 404 502 = @process;
    	}

    	# 如果请求内容不在缓存中，则请求指向这里
    	location @process
    	{
    		proxy_pass http://backend;
    		ssi on;
    	}
    }
    # 主服务器
PHP
需要两个脚本，authentication.php 和 somepage.php，实际上是同一个页面，应该看起来类似下面这个样子：

    <?
    $memcache = new Memcache();
    $post = posts::get($_GET['id']);
    ob_start();
    ?>

    <h1><?=$post['title']?></h1>
    <p><?=$post['html']?></p>
    <br/>

    <?
    $html = ob_get_clean();

    $memcache->set($_SERVER['REQUEST_URI'], $html);
    echo $html;
    ?>

结论：SSI在服务器层面增加了缓存使用的灵活和便捷性，可以让我们充分缓存动态站点。
