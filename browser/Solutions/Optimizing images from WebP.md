Optimizing images from WebP

Minification and compression have long become quite standard things for optimizing the code of web pages. All popular web resources optimize images , use the same CSS when possible, and choose the right image formats . WebP vs JPEG

But this is not enough. Statistics HTTP Archive shows that images occupy about 64% of the size of a web page. The new standard WebP, capable of replacing JPEG and PNG, comes to the rescue.

Briefly about WebP

The format appeared in 2010 and has since been developed by Google. WebP is based on the algorithm for compressing the key frames of the VP8 video codec with or without loss, and is packaged in a container based on RIFF. WebOS loseless images are 26% smaller on average compared to PNG, and WebP lossy is 25-34% less than JPEG with the same SSIM index . The new format also supports transparency (known as alpha channel).

Principle of operation

Lossy compression in WebP uses predictive coding, in which the values ​​of neighboring pixel blocks are used to predict the value of the desired pixel block, and then the difference is encoded.

Lossless-compression uses already known fragments of the image to accurately reconstruct the pixels. A local palette can also be used if there is no matching algorithm.

Advantages and disadvantages

Behind:

smaller image size;
advanced compression algorithm;
high image quality;
support for transparency.
Against:

poor prevalence;
"Plasticity" when compressed with loss;
The colors in pixel and other computer graphics can be lost.
WebP is already supported in Chrome, Opera and the standard Android browser, and using the WebPJS library can be displayed in all popular browsers (in IE 6 and higher with Flash).

In addition, a light libwebp encoding and decoding library , a command-line utility for encoding and decoding WebP, as well as tools for viewing, multiplexing and animating images in this format are developed .

Installing utilities and converting to WebP

All tools can be downloaded from the Google Developers page . They exist for Windows, Linux and MacOS X in compiled form, but you can also download the source code for development (opensource yet) or self-compilation .

To convert from JPEG, PNG and TIFF, the utility cwebp is used , and for decoding - dwebp .

Conversion is started by a simple command (from the directory with utilities):

cwebp input.png -q 80 -o output.webp
# Specify the names of input and output files, quality (from 0 to 100)

The same principle starts decoding. There are many options and additional parameters, including for checking the encoding.

Deploying WebP

So, you are interested in the new format, you conducted all the tests, again looked at the statistics and made sure that Chrome is still the most popular web browser. What next?

Next, you need to make a copy of all the images in WebP (you can write a simple script to convert all the files), and then check the users of the site and upload compact images to them if their browser supports WebP. Accept negotiation Nginx

That is, you can create your own script, which will check the client's browser to support the format, which will then pop up the web server, or completely assign this task to the web server. The second option seems more logical to us.

Reconciliation with the Accept header

Browsers pass the Accept header in the form:

 # в Opera
Accept: text/html, application/xml;q=0.9, application/xhtml+xml, 
image/png, image/webp, image/jpeg, image/gif, image/x-xbitmap, */*;q=0.1

 # в Chrome
Accept: image/webp, */*;q=0.8
# Opera displays all supported formats, Chrome also specifies WebP support

Knowing this, you can configure the web server to automatically transfer the WebP. As an example, we use Nginx, which should be added to the configuration file:

location / {

  # проверка заголовка Accept и наличия версии файла в .webp 
  if ($http_accept ~* "webp")    { set $webp_accept "true"; }
  if (-f $request_filename.webp) { set $webp_local  "true"; }

  # если WebP есть, то передать Vary
  if ($webp_local = "true") {
    add_header Vary Accept;
  }

  # если клиент поддерживает WebP, то передать файл
  if ($webp_accept = "true") {
    rewrite (.*) $1.webp break;
  }
}
# The Apache configuration will be similar

That is, if Accept does not find support for WebP, then ordinary files are transferred.

Well, if Nginx is used as a proxy for statics caching, then the configuration will be different:

server {
  location / {
   
    if ($http_accept ~* "webp") { set $webp T; }
    proxy_cache_key $scheme$proxy_host$request_uri$webp;

    proxy_pass http://backend;
    proxy_cache my-cache;
  }
}
# Check for WebP indicator and redirect to remote servers

The most important thing

The format of WebP images will significantly reduce the size of the web page, but given its limited support, you must additionally configure the web server and keep copies of all images in several formats.