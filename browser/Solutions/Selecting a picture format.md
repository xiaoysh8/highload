Selecting a picture format

In many cases, the total size of the images that are loaded on the page is more than 50% of the weight of all pages. It is critically important to select the correct format for the images used. There are only two PNGs and JPGs, so it's not that hard.

PNG

PNG format
PNG is a very cool format that has replaced GIF. Format features:

Transparency. If transparency is needed, choose this format immediately.
Different palette (8 and 24 bits). Allows to make small on color scale pictures small in size.
When to use?

PNG allows you to display images without losing the smallest details and with accurate color rendering. This makes the format convenient for:

icons
small-color illustrations
images, which require greater clarity of small parts (the size of such an image can be huge, so it is better to avoid such situations)
Palettes PNG24 and PNG8

The PNG24 format uses the maximum color depth. This type is suitable for multi-color pictures. PNG8 allows you to use a limited palette from 1 bit (2 colors) to 8 bits (256 colors). This can significantly reduce the size of files with few colors in the picture.

Convert PNG24 to PNG8 using pngnq :

pngnq -n 256 image.png
# Convert to PNG8

JPEG

JPG format
This format uses the most accessible palette. To reduce the size, a special compression and anti-aliasing mechanism is used.

When to use?

The strong side of the JPEG format is manifested when the image contains many colors and there are no special requirements for small details. This is ideal for:

photos
screenshots
multicolor illustrations
Progressive JPEG

JPG supports a progressive format. When a person opens such a picture in the browser, he first sees the general outlines, and then the detail and quality will increase to the maximum. This will give the impression of a faster download of the site. It is especially important to use the progressive format for cases with a low speed of Internet access for visitors.

Conversion to a progressive format can be performed with the help of jpegtran :

jpegtran -progressive -outfile image.jpg image.jpg
# change the picture format to progressive

Either imagemagick:

convert -interlace Plane input-file.jpg output-file.jpg 
# change the picture format to progressive

You can check the use of the Progressive format with this tool .

Smoothing

JPEG allows you to specify the level of compression / smoothing when saving an image. This reduces the quality. But sometimes the quality is reduced imperceptibly, but the size savings can be quite large. To change the compression, use the jpegtran utility :

jpegtran -quality 80 -outfile optimized.image.jpg image.jpg
# Use values ​​from 5 to 95 to get different compression levels

GIF

It makes no sense to use the GIF format, because there is a PNG. One case where you want GIF is the presence of an animation. Can be used, for example, for preloaders .

General recommendations

As the most important - Checklist for working with pictures on the Web.