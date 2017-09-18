Google AMP for accelerating content pages

Google new initiative aimed at improving the web experience of mobile users. Mobile Pages for Accelerated - a new format of web pages built for optimizing content (images, JS, social plugins Networks.) And an increase of responsiveness of the mobile version of the site, in fact facilitating page. Google AMP HTML

principle of operation

AMP consists of three parts:

The HTML the AMP - slightly modified the markup language in which some of the tags are replaced by equivalents, with a focus on the AMP, and a part of prohibited altogether;
JS AMP - AMP restriction on the use of the library, which is tuned for asynchronous loading;
Google the Cache AMP - AMP optional caching features pages on the Google server for quick user feedback.
Basic requirements and restrictions:

Only asynchronous scripts;
Only built-in CSS;
Styles limited size 50K;
You must specify the size of HTML-links to external resources (images);
JS is always performed asynchronously;
It supports only JavaScript to AMP;
Fonts downloaded using the link tag or the CSS @ font-face rule .
AMP HTML

I analyze a simple HTML-pages that are based on AMP requirements:

<!doctype html>
<html amp lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello, AMPs</title>
    <link rel="canonical" href="http://example.ampproject.org/article-metadata.html" />
    <meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
    <script type="application/ld+json">
      {
        "@context": "http://schema.org",
        "@type": "NewsArticle",
        "headline": "Open-source framework for publishing content",
        "datePublished": "2015-10-07T12:02:41Z",
        "image": [
          "logo.jpg"
        ]
      }
    </script>
    <style amp-boilerplate>body{-webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;animation:-amp-start 8s steps(1,end) 0s 1 normal both}@-webkit-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-moz-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-ms-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-o-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}</style><noscript><style amp-boilerplate>body{-webkit-animation:none;-moz-animation:none;-ms-animation:none;animation:none}</style></noscript>
    <script async src="https://cdn.ampproject.org/v0.js"></script>
  </head>
  <body>
    <h1>Welcome to the mobile web</h1>
  </body>
</html>
# AMP page with new tags and asynchronous script

AMP-Syntax

HTML-document developed by social requirements, should follow the syntax :

Begin with < the Doctype the html!> ;
Contain the top-level tag < the html ⚡> or < the html amp> ;
Contain the < head> and < old body> ;
First in the < head> should be < the meta the charset = "utf-8"> ;
Contain a reference < link the rel = "canonical" the href = "$ some_url" /> within the < head> , which indicates the normal version of the page;
Contain < the meta name = "the viewport" content = "width = device-width, minimum-scale = 1"> within the < head> ;
Contain <style amp-boilerplate> within the < head> ;
Before the closing < / head> should be 
< script the async the src = "https://cdn.ampproject.org/v0.js"> < / script> , which includes a JS library for AMP.
Additionally, the < / head> can include styles :

<style amp-custom>
  # Здесь размещается нужный стиль
  body {
    background-color: white;
  }
  amp-img {
    background-color: gray;
    border: 1px solid black;
  }
</style>
# Includes a custom style

The main component of the Integrated AMP HTML

ad-amp - container for advertising blocks.
It supports only advertisements on the channel the HTTPS . It requires a script:

<script async custom-element="amp-ad" src="https://cdn.ampproject.org/v0/amp-ad-0.1.js"></script>

<amp-ad width=300 height=200
      type="adsense"
      data-ad-client="ca-pub-8125901705757971"
      data-ad-slot="7783467241">
  </amp-ad>
# Set the AdSense ads

Amp-ad also supports built-in styles and a number of advertising services .

img-amp - managed by the runtime replacement for the standard tag < img> . It supports styles. Used in the form of:
 <amp-img
            src="img/hero@1x.jpg"
            srcset="img/hero@1x.jpg 1x, img/hero@2x.jpg 2x"
            layout="responsive" width="360" placeholder
            heights="(min-width:1420px) 20%, (min-width:1320px) 25%, (min-width:1000px) 40%, (min-width:760px)  85%, (min-width:500px) 100%, 120%"
            alt="an image"
            height="216" on="tap:headline-img-lightbox" role="img" tabindex="0">
        </amp-img>
# Inserting a picture showing the different sources for different screen sizes

pixel-amp - element to track page views. Posts: it does not support styles, the code is as follows:
<amp-pixel src="https://foo.com/pixel?RANDOM"></amp-pixel>
# Indicates trekking, randomly generated code

video-amp - replacing the tag < video> from HTML5. Note that the component can only be used for direct insertion of HTML5 video:
<amp-video width=400 height=300 src="https://yourhost.com/videos/myvideo.mp4"
    poster="video-poster.jpg">
  <div fallback>
    <p>Your browser doesn’t support HTML5 video</p>
  </div>
  <source type="video/mp4" src="foo.mp4">
  <source type="video/webm" src="foo.webm">
</amp-video>
# Indicates the size and path of the video, as well as an error if HTML5 is not supported

To insert a YouTube video using a separate script and separate tag:

<script async custom-element="amp-youtube" src="https://cdn.ampproject.org/v0/amp-youtube-0.1.js"></script>

<amp-youtube
      data-videoid="WHD8FM2lpUE"
      layout="responsive"
      width="480" height="270"></amp-youtube>
ID # Insert video from YouTube, indicating the option to adjust the window size

AMP supports the extension of HTML to insert clips from FaceBook, photos from Instagram, payment, analytics, animation, audio, and other popular features.

rel = "canonical"

If a site has two versions of the page, standard and AMP, then for proper indexing in Google must specify the cross-reference:

 # На основной странице
<link rel="amphtml" href="https://www.example.com/url/to/amp/document.html">

 # На AMP странице
<link rel="canonical" href="https://www.example.com/url/to/full/document.html">
# Search Engine automatically indexes the AMP-version with indexing main page

AMP-pages Checking

There are two ways to check AMP pages for errors. The first and easiest - to add to the URL-address of the new page # development = 1 in the form of:

http://www.example.org/released.amp.html#development=1
# Errors are displayed in the console for developers of Chrome DevTools

In addition, the company has developed a console validator (in the status Beta) , written in JavaScript:

$ ./index.js testdata/feature_tests/minimum_valid_amp.html
testdata/feature_tests/minimum_valid_amp.html: PASS

$ ./index.js testdata/feature_tests/several_errors.html
testdata/feature_tests/several_errors.html:23:2 The attribute 'charset' may not appear in tag 'meta name= and content='.
testdata/feature_tests/several_errors.html:26:2 The tag 'script' is disallowed except in specific forms.
# Displays errors in the HTML

AMP Cache

To speed up the return AMP page, use another Google service - AMP Cache. To distribute content using CDN, you need to alter their URL:

 # Было 
https://example.com/amp_document.html

 # Стало
https://cdn.ampproject.org/c/s/example.com/amp_document.html
Prefixes # / c / s is a protected compound, / c - normal

Source URL address may also include a query string. Google provides AMP URL API to automatically retrieve cached pages using batchGet method.

most importantly

If a significant part of the audience comes from mobile devices, the creation of AMP-versions of your pages will improve user interaction with the site. Do not forget about optimizing your web server and mobile optimization .

den Golotyuk