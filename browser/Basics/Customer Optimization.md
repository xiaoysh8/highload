Customer Optimization

Client optimization is a set of techniques that make your site faster for the user without significant changes on the server side. Increasing speed can sometimes achieve a tenfold improvement. In this case, the techniques are quite simple.

Causes of low speed

What happens when a user opens a site? The browser sends a request to the server, receives the content of the site and shows it to the visitor: Request from browser to server

This simple case is typical for a site without pictures, styles and scripts. Today it is almost impossible. Opening a site that uses modern technology, looks like this: Multiple requests from the browser to the server

How to increase the speed?

It is clear that the speed of the site depends on:

the number of additional requests (JS, CSS, pictures).
the amount of data transferred from the server to the visitor.
All methods of accelerating Web sites are aimed at reducing the number of requests and reducing the amount of data.

Reducing the number of requests

Client caching

Client caching
Important Install the HTTP headers Expires and Cache-control so that your CSS / JS files are cached on the client browser. Thus. the client will only request unique files once for the duration of the cache action.

Gluing

Bonding the statics
Paste all CSS files into one. The same thing should be done for JS. Then you will have only two external files: 1 javascript and 1 CSS.

Loading external resources

The basic rule is to load any external resources as close as possible to the end of the HTML. The only exception is CSS, it needs to be loaded in the <head> tag . Javscript needs to be loaded before the closing </ body> tag . First, download Javascipt for the application and then counters.

CSS Sprites

CSS Sprites
A large number of small images on one page will lead to a large number of requests to the Web server. Use CSS sprites . Small pictures on the site should be merged into one. And for display, you need to use the "background" property in CSS.

Reducing the amount of data

Compression

compress gzip
Important Almost all modern browsers support the compression of gzip or deflate. Compression makes sense only for text files (HTML, CSS, JS). Compression can be included both on the Web server and in the application itself, for example in Nginx:

server {
...
gzip on;
gzip_disable "msie6";
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
...
}
All known Web servers have modules for compression .

Compression can also be used in applications instead of the Web server. For example, in PHP, compression is included in php.ini:

zlib.output_compression On;
Image Optimization

Image format
Important Images can occupy more than 50% of the entire page size. Compressing images without loss of quality can sometimes save several hundred kilobytes. There is a whole bunch of tools for lossless compression.

Image format

Pay attention to the choice of the format of pictures:

JPEG for photos
PNG is better for icons and illustrations
More about choosing the correct picture format.

Minification

Minify
Use the minify mechanism to get a more compact JS and CSS size (and HTML ). The very process of minimization is very simple. This is cutting out all comments and optional white space characters (hyphens, tabs, etc.).

Correct HTML

The size of modern pages can be very large because of the complex layout and a large number of elements. The more the HTML the worse. Therefore, observe the rules:

Do not use built-in styles (style = "...")
Bring all styles to external CSS files
Bring Javascript to external files
Use asynchronous Javascript loading
Use short, but understandable class names
Use standard markup elements to highlight text (strong tags, em, i, u) instead of creating extra classes
The most important

Correct use of client caching and gzip compression will give you 80% of the result. If you have a lot of pictures on the site, their optimization can also bring significant benefits.