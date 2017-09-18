Cache-control with dynamics

The Cache-control header allows you to significantly increase the download speed of the site, as well as to offload the channel between the server and the user. This header includes caching and is easy to use for files that never change. But in order to use this advantage for files that can change , you need to better understand the client caching:

The server sends an additional header Cache-control
The browser sees this header and saves the file to its cache
On the next request to the file (for example, the visitor moved to another page, and there is a download of the jQuery library again), which is in the cache, the browser will not send a request to the site. It shows the locally saved version
Repeated request to the site to download cached files the browser will start sending only when the date specified in the header of the Cache-control (it can be configured)
Browser cache
Enable caching

First, we need to enable caching for JS / CSS on the server (we have Nginx):
server {
...
location ~* ^.+\.(js|css)$ {
	expires max;
}
...
}
Using Versions

But in the main application, you need to add the files of the so-called JS / CSS files to the download path. version:

<link rel="stylesheet" href="/styles.css?r4" type="text/css" />
<script type="text/javascript" src="/scripts.js?r7">
Where "r4" and "r7" are just the numbers that you specify (the version of the file, it's best to start with 1). With each update of files, you just need to change its version (increase by 1). For example, after some changes to styles.css we will increase its version:

<link rel="stylesheet" href="/styles.css?r5" type="text/css" />
# The new version will force the browser to download a new file, because the path to it has changed (for the browser it's a new file)

The most important thing

Caching allows you to significantly accelerate the loading of web pages. A few simple steps will help cache dynamic content.