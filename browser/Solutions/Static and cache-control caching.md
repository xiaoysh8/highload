Static and cache-control caching

Client caching is the ability of the browser to store files locally, so that they do not need to make repeated calls to them. This is very useful for images and CSS / Javascript files. When a person navigates through the pages, the browser will constantly request the same files if you do not use the caching mechanism on the browser.

Control what files should be cached using the HTTP headers Cache-control and Expires. The server sends such a header along with the response, telling the browser whether or not to save the file to the local store. Cache-control header

If the file was saved in the cache, then the next time the file is accessed, the browser will receive its contents locally. Thus, everything will happen much faster, because there will be no request to the server. Client caching

Cache-control

In order to control caching in the browser, the HTTP header Cache-control is used. It must be transferred with all the files that need to be cached. It has the following format:

Cache-Control: private, max-age = 0, no-cache
private means that caching will only work on the user's browser. Instead, you can use the public statement . This allows caching on public proxy servers (such often exist in companies).
no-cache means that this request can not be cached.
max-age is the time for which the result will be cached. It is set in seconds.
Usually this header is enough to make everything work:

Cache-Control: private, max-age = 60
# cache the query result in the browser for 60 seconds

Cache-Control: private, max-age = 0, no-cache
# disable query caching

Expires

The optional HTTP Expires header specifies the date and time when the browser should update the cache:

Expires: Thu, 31 Dec 2037 23:55:55 GMT
# The browser will send a repeated request already in 2037, before that time it will use the cache

This header should be used in conjunction with Cache-Control.

Vary

The Vary header allows you to set additional rules for caching queries:

vary: User-Agent
# The browser will know that the content may vary depending on the version of the site (for example, mobile and desktop)

What to cache

Cache all files that change less frequently than every few minutes. Without fail:

Pictures
CSS
Javascript
Downloadable files (archives, documents, etc.)
In-app use

In the application, the Cache-Control headers are usually not used, because applications generate dynamic content. If you rarely change the site, you can add caching to reduce the number of requests to the server. For example, in PHP:

<?
header("Cache-control: public");
header("Expires: " . gmdate("D, d M Y H:i:s", time() + 60*60) . " GMT");
# Turn on the cache for 1 hour

However, it is better not to do this, because When changing content, different visitors will have different versions of the pages.

Use on the Web server

Headers for images and static files (JS / CSS) should be installed on the Web server.

Nginx

In Nginx, caching is enabled with the expires instruction:
server {
...
location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
		expires max;
	}
...
}
# Enable the cache for an infinite period for files with the listed extensions

Apache

In Apache, caching is enabled by the module mod_expires and looks like this:

<FilesMatch "\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$">
    Header set Cache-Control "max-age=2592000, must-revalidate"
</FilesMatch>
# Enables cache for files for 1 month

Variable files

Javascript and CSS files usually change. But it is not possible to determine in advance when it will be necessary to make a change. You can cache such files for a short time (for example, a minute), but this will not allow you to take full advantage of Cache-Control.

The browser caches files according to their URLs. If we add the GET parameter to the path, the path will change and the browser will request the file again. Which is what we need. With each update of CSS and Javascript files, you should add a new value to the GET parameter: Browser Cache

It is best to use consecutive numbers ( versions ), and with each change simply increase them by one:

<script src="/jquery.js?5"></script>
The most important

Client caching can increase the speed of your site several times. Be sure to use this feature. To check the correct use of Cache-control of any page, you can use the Cache-control Checker tool .