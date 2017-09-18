Optimization in HTTP / 2

The HTTP 1.1 protocol served as faith and truth for almost 20 years, so the appearance of the new specification was only a matter of time. It was replaced by HTTP / 2, officially adopted last year and based on the SPDY protocol developed by Google.

If you still doubt whether it is worthwhile now to switch to HTTP / 2, then the answer is: yes, it's worth it. First, all new versions of popular web servers already support the protocol.

To be included in Nginx (version 1.9.5 or higher) in the configuration file ( server section ), the listen directive should be added :

server {
    listen              443 ssl http2;
    server_name         example.com;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
}
# Nginx supports HTTP / 2 only paired with TLS

Well, in Apache (starting with version 2.4.17 and above), you need to complete the configuration file:

# for a https server
Protocols h2 http/1.1


# for a http server
Protocols h2c http/1.1
# HTTP / 2 support for secure and unprotected connections

Secondly, all new versions of the most popular browsers also support HTTP / 2.

And thirdly, the new specification is much faster and more productive than HTTP 1.1. In addition, more than 7.1% of websites on W3Techs data already support HTTP / 2.

The main differences between HTTP / 2 and HTTP 1.1

The main differences between the protocols are a few.

Binary

Unlike text HTTP 1.1, HTTP / 2 is binary. Therefore, the protocol is more efficient for parsing, more compact in transmission, subject to fewer errors.

Multiplexing

In HTTP 1.1, browsers use multiple connections to the server to load a web page, and the number of such connections is limited. But this does not solve the problem of blocking the channel with slow packets. While HTTP / 2 uses multiplexing, which allows the browser to use one TCP connection for all requests. HTTP / 2 multiplexing

All files are loaded in parallel. Queries and responses are divided into frames with meta-data that associate queries and responses. So they do not overlap each other and do not cause confusion. The answers are obtained as they are ready, therefore, heavy queries will not block processing and issuing simpler objects.

Prioritization

Along with multiplexing, traffic prioritization appeared. Queries can be prioritized based on importance and dependency. HTTP / 2 Prioritization

So when you load a web page, the browser will first receive important data, CSS code, for example, and all secondary processes will be processed last.

HPACK header compression

The HTTP protocol is built in such a way that when sending requests, headers that contain additional information are also transmitted. The server, in turn, also attaches headers to the responses. And given that web pages consist of many files, all the headers can occupy a decent amount. Therefore, in HTTP / 2 there is header compression, which will significantly reduce the amount of auxiliary information, so that the browser can send all requests at once.

Server Push

When using the HTTP 1.1 protocol, the browser requests a page, the server sends HTML to the response and waits for the browser to process it and request all the necessary files: JavaScript, CSS and photos. Therefore, the new protocol introduced an interesting feature called Server Push. HTTP / 2 Server Push

It allows the server immediately, without waiting for the response of the web browser, to add the files it deems necessary to the cache for quick delivery.

Encryption

The HTTP / 2 protocol does not require encryption of the channel. However, all modern browsers work with HTTP / 2 only together with TLS , like Nginx. So the massive implementation of the protocol should contribute to the spread of encryption on the Web.

Therefore, if you already use TLS, then you should use HTTP / 2, which reveals the full potential of encryption. When creating an encrypted connection, only one TLS Handshake occurs, which greatly simplifies the entire process and shortens the connection time.

Optimizing HTTP / 2

The main optimization of HTTP / 2 compared to HTTP 1.1 is the deactivation or modification of many optimizations of the previous version of the protocol.

It is worth to abandon the domain shadding. This way of distributing multiple files across different domains and CDN is relevant for HTTP 1.1, as it solves the problem of parallel connections. But in the case of a new protocol, this decision degrades performance and eliminates the prioritization of traffic.
If possible, refuse or modify sprites. Combining a lot of small pictures into one large image can increase the speed of page loading, but if a user visited a web page with one small picture, then he still sent the entire sprite. In the case of HTTP / 2, such a solution will be less useful in view of the appearance of multiplexing.
Another method for optimizing images is to embed with DataURI. It can also be useful in HTTP / 2, but it will definitely be less efficient than in the case of the previous version.
It is better to refuse from concatenating files. The method is similar to sprites - all the necessary files, CSS and JavaScript, are combined into one big one for transferring one thread by one connection. So if a user visits a page with one small JS code, he will still be sent the entire merged file. Another difficulty is that all the merged files need to be unloaded from the cache simultaneously, and one change in the code of any of them requires updating the whole set. So, thanks to the same multiplexing, this approach does not make sense.
Also, you should not embed files in HTML code.
The most important thing

The HTTP / 2 protocol has already been significantly optimized, compared to HTTP 1.1, so simply implementing a new specification can improve the performance of web services. And disabling the additional tricks that were used to accelerate HTTP 1.1 will take advantage of all the advantages of HTTP / 2.