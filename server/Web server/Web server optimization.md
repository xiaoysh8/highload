Web server optimization
=======================

>web服务器是所有网站的主要应用层之一。它从客户端接受请求，生成并发送响应。随着请求数量的增加，web服务器的性能随之下降。

-------------
![](https://i.onthe.io/smngoz12tmm0s7uv8.2d6f671f.jpg)


基本优化
--------
有几个简单方法可以提高服务器性能，如下：

- 压缩请求数据
- 减少请求数量
- 增加/减少需要的请求资源

####压缩请求数据
![](https://i.onthe.io/smngoz75prvkp1ofn.ab7d2b98.jpg)

现代浏览器都支持gzip压缩。服务器先压缩响应内容，然后再向客户端发送。浏览器先进行解压，然后进行渲染显示。通过一个过程，文件体积可以减小70%。web页面的加载速度更快。

只有文本类型的文件可以被压缩，这一操作也会消耗服务器一些资源，很少，可忽略不计。

这个[工具](http://highloadtools.com/gzip "是否启用gzip压缩")看看你的服务器有没有使用gzip压缩。

####减少请求数量
![](https://i.onthe.io/smngozq7fbj14cjrg.8ffc1c6e.jpg)

web页面上的每一个图片或脚本文件，需要服务器提供一个单独的进程来进行处理。一个页面拥有10张图，3个脚本文件，那每个访客这一页就要进行14次请求。

浏览器单页面需要请求的数量越少越好，可以按照以下原则来实现这点：

-	[将css和js文件进行合并](http://thehighload.com/post/Minify+your+js%2Fcss%2Fhtml)，减少外部资源请求
-	使用浏览器端缓存。让浏览器不必重复下载同样的资源文件。要启用这一功能，web服务器要发送一个特殊的缓存控制头(Cache-Control header )给浏览器。

[查看一下我们的网站有没有启用这个功能？	](http://highloadtools.com/cachecontrol)

####配置参数
![](https://i.onthe.io/smngoz16l4jdk7uti.0a240ae4.jpg)

没有恰当配置参数的服务器，基本上就无法发挥出服务器所有的潜力和资源。通过调整配置，性能可以提高几个档次。

每一种web服务器软件都有自已一套配置选项，需要分别来配置:

-	[优化nginx](http://thehighload.com/post/Optimizing+Nginx+configuration)
-	[优化apache](http://thehighload.com/post/Apache+server+tuning/)


要点
----
-	压缩且减小请求的数据体积和数量，可以提高响应速度达90%
-	优化服务器配置，可以在保持服务器性能的前题下，增加更多的在线人数。
