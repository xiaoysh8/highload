Mobile Optimization

General rules for optimizing mobile applications are the same as for desktop sites. Nevertheless, there are a number of additional recommendations that will make the mobile version faster.

Remove render-blocking elements

Render-blocking elements
Render-blocking elements are any external Javascript and CSS files that are loaded in the usual way (not asynchronously):

<link rel="stylesheet" href="/styles.css" type="text/css" media="all" />
<script type="text/javascript" src="/scripts.js"></script>
The browser is forced to wait for a response from such resources before displaying the page to the user. Use asynchronous loading of Javascript and CSS .

Do not use plugins

Flash
Do not use Flash and applets, many mobile browsers do not support them. HTML5 contains everything you need to work with media formats.

Set up the viewport

Viewport
Use the viewport to adjust the width of the screen for mobile devices:

<meta name="viewport" content="width=device-width, initial-scale=1.0">
Font size

Font size on mobile devices
Make sure the text size matches the size of the display. It should not be too small to be readable on a mobile device.

Size of controls

Size of controls on mobile devices
The dimensions of the controls should be large enough that they can be pressed on tap-devices.

Pagespeed Analysis

Analyze using Google Pagespeed to get recommendations about the mobile version of your site.