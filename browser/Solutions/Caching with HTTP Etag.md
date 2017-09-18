Caching with HTTP Etag

Etag (or entity tag ) is one of the caching mechanisms in HTTP. In fact, this is an identifier that is assigned to the file by the server for later verification. http etag

When a client requests web page files (images, CSS, etc), the server transfers all the data along with the Etag tags in the form:

HTTP/1.1 200 OK
Server: MyServer/2.1
Date: Thu, 09 Jun 2016 13:30:54 GMT
Content-Type: text/html
Accept-Ranges: bytes
Last-Modified: Tue, 07 Jun 2016 12:00:00 GMT
ETag: "6d82cbb050ddc7fa9cbb659014546e59"
Content-Length: 363

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
...
</html>
# Example of server response with Etag

The files are stored in the cache on the client side, and when the browser requests these files again, the If-None-Match line is added to the query :

GET /news/latest.html HTTP/1.1
If-None-Match: "6d82cbb050ddc7fa9cbb659014546e59"
Host: example.com
# Sample request with Etag

The server, in turn, checks Etag, if it matches, then the server sends the code 304 :

HTTP/1.1 304 Not Modified
Server: MyServer/2.1
Date: Tue, 07 Jun 2016 09:00:00 GMT
ETag: "6d82cbb050ddc7fa9cbb659014546e59"
Content-Length: 0
# Indicates that the files have not changed and can be taken from the cache

Otherwise, the requested files will be sent again.

Enabling Etag

All modern web browsers and web servers support HTTP Etag. To enable the function in Nginx, you need to edit its configuration file:

server {
...
	location ~* ^.+\.(rss|atom|jpg|jpeg|gif|png|ico|rtf|js|css)$ {
		expires 2592000;
etag on;
	}
...
}
# Etag is enabled for static files

The most important thing

HTTP Etag allows you to increase the responsiveness of the web application and reduce the load on the channel. The function is supported by all modern web browsers and does not require additional configuration after power-up.