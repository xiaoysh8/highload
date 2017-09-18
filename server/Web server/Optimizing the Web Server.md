Optimizing the Web Server

Web server is the very first link in the operation of any Web site. It receives a request from the client, generates a response and sends it back to the client. When the number of such requests increases, the speed of the Web server will fall. Web Server Operation

Basics of optimization

There are several simple methods for increasing the efficiency of the Web server. Everything is based on three principles:

Query Compression
Reducing the number of requests
Increase / limit resources
Query Compression

Gzip reduces file size
All modern browsers support working with Gzip compression. The Web server compresses the contents of the response before sending it to the client, and the browser extracts it at the time it is received. So you can save up to 70% of the file size. For customers, this will mean a higher speed of the Web site.

Compression works only for text format files (HTML / XML, CSS, Javascript). Compression will also lead to additional load on the Web server, but insignificant.

How can I check if gzip compression is enabled?

Reducing the number of requests

Requests from clients to the Web server
Each picture or script on the Web page is a separate request to the Web server. 10 pictures and 3 Java scripts on the page will result in the Web server receiving 14 requests from each visitor:

1 basic query + 10 pictures + 3 JS = 14
The less each client will send requests to the server, the better. There are several simple approaches for this:

Minimize the number of external requests from the page (gluing css / js, CSS sprites).
Use client caching so that the client does not send repeated requests to the server. To do this, the Web server must send a special Cache-control header to the browser. How can I check if Cache-control is enabled?
Setting up resources

Configuring Nginx Resources
A web server without internal configuration most likely does not use all available platform resources. Adjusting the parameters can increase the efficiency of the work several times.

Each Web server has its own set of parameters and is configured individually:

Detailed description of the Nginx configuration for optimal performance
Detailed description of Apache tuning for better performance
The most important

Compressing and reducing the number of requests to the Web server will bring 90% of the effect to your visitors
Optimizing server settings will help increase the number of visitors it can serve without loss