Google PageSpeed ​​for website speed analysis

To analyze the speed of the site, it's best to use the Google Pagespeed tool . It will determine the site's relevance to recommendations and will show current problems.

To conduct the analysis, simply enter the URL of the desired page of the site. After that you will see a list of recommendations.

Consider the recommendations and importance of each of them for visitors to your site.

Leverage browser caching

Very important Client caching
If you do not use client caching , you should start with this. As it is done for different servers and applications, see the materials for Cache-control .

Optimize images

Very important Pictures
Correct choice of the format of pictures and their compression can reduce the size of the data received by the client several times. Be sure to use picture compression tools .

Enable compression

Very important gzip
The compression of gzip reduces the size of the text data received by the client. Savings can reach 70%. All modern browsers (including mobile browsers) support compression; it is part of the HTTP 1.1 protocol.

Eliminate render-blocking JavaScript and CSS in the above-the-fold content

Desirable for landing pages Render-blocking JavaScript and CSS 
Any external Javascript or CSS call pauses loading the page until the response (css or js file) is received. This is not for the landing pages, tk. this will result in a slower page load. Landing pages should use asynchronous Javascript and built-in styles .

Avoid landing page redirects

Desirable Redirect
Try to avoid redirects from landing pages. These are the pages from which the visitor starts using your site. Redirects increase the time during which a person will have to wait for the site to load.

Minify CSS / JS / HTML

Desirable Minify
Minimizing Javascript, CSS and HTML is a special technique for removing unnecessary characters from the code (spaces, tabs, and line breaks). Sometimes this saves up to 20% of the file size. YUI compressor is a convenient and simple solution for minimizing statics.

Prioritize visible content

It is necessary to pay attention Prioritize content
Prioritization of Above-the-fold content is important for large pages. Once the browser has received a piece of HTML code, it will try to show it to the visitor. If some elements of the first screen are at the end of the HTML code, this will cause the browser to load the entire page first, and only then show it to the user. It can be critical for mobile devices when a visitor uses a slow communication channel.

Reduce server response time

It is necessary to pay attention Response time from the server
The speed of page generation (ie the application itself, for example, PHP) usually does not have a significant effect on the speed of the site for the user. If it is within 300 ms. If the generation takes a second or more, you should optimize the server part .

The most important

Use PageSpeed, the first three recommendations can give an increase in the speed of the site several times. Read in detail about client optimization and site acceleration techniques.