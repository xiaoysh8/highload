Banner advertising

Any banner system affects the speed of the site. Banner systems usually use highly non-optimized solutions, including:

a bunch of redirects
uncompressed pictures
uncompressed javascript 's
synchronous load codes
All this can significantly slow down the site. But developers usually do not have any influence on the work of banner systems (unless one uses something of their own). There is a hack that will reduce the effect of slow banners.

Asynchronous ad loading

It would be nice to first load the entire page for the user, and only then to draw the banner. But this option is not available for all systems of unscrewing. For example, Google Adsense has made support for asynchronous downloads relatively recently.

Most banner codes use the Javascript call to document.write (). This makes it impossible to directly run such code on the body.onLoad () event. Therefore, it is not possible to simply transfer the code to an asynchronous script.

Iframe

The easiest way to do asynchronous loading of ads is to transfer it to the iframe. The contents of the browser frame loads regardless of the main site, so this will not slow down its download in any way.

Hidden Downloads

Hidden advertising download
If there is no possibility to use iframe, then you can take advantage of the good feature of modern browsers. Namely - download the hidden content in the last place. Display the picture in a hidden block:

<div style="display: none"><img src="/ad.png"/></div>
It will be loaded after those elements that are visible. In some browsers, the image will not be loaded at all until the block becomes visible. Just what we need. To apply this to advertising it is necessary:

1. Show all banners in a hidden container

Turn all ads in display: none element:

<div style="display: none;" class="banner"><script>Тут баннерный код</script></div>
# This container should be used for all banners

2. We show blocks with ads after downloading content

Using jQuery:

<script>$(document).ready(function() { $('.banner').show(); });</script>
# Show all ads after downloading the document

The most important

This little trick can reduce the site's dependence on slow advertising. Worth a try.