Optimizing images on the Web

Images can take up to 90% of the total data on the page. Their correct optimization can significantly affect the speed of the site.

How to optimize pictures?

Often, image files store a lot of additional information, such as details about taking photos, comments, text descriptions, geographic data, etc. When used on the Web, this data is not needed. There are tools that allow you to cut out all unnecessary meta data, which reduces the size of the picture. Image Optimization

Formats

Correct choice of the image format will allow you to get good image quality with the minimum size. You should use JPEG for photos and PNG for icons.

Instruments

The toolkit for optimizing images is aimed at lossless compression , because compression without loss of quality. You can also adjust the compression level for different formats yourself to get even more benefit.

imagemagick

Modern cameras usually record a bunch of metadata in the JPEG photos (geolocation, preview, etc.). If you cut them, you can sometimes save a few tens of kilobytes. Convert from imagemagick contains a useful parameter "-strip" for cutting out all unnecessary metadata from the JPEG image:

convert test.jpg -resize 100x100 -strip test_100.jpg
Jpegtran

Jpegtran allows you to cut out metadata and perform lossless optimization JPEG:

jpegtran -copy none -optimize -outfile min.image.jpg image.jpg
# save the optimized copy to min.image.jpg

Jpegoptim

Another useful utility jpegoptim for optimizing JPEG:

jpegoptim file.jpg
# Optimizes the file and stores it in itself

Optipng

The tool for optimizing PNG images optipng works like this:

optipng test.png
# Optimizes the file and stores it in itself

Pngcrush

Pngcrush works like this:

pngcrush -reduce -brute in.png out.png
# Optimizes in.png and writes the result to out.png

The most important

Be sure to use picture compression tools without losing quality. It takes a minute, but can reduce the size of the images several times. Of the tools, it's best to use Pngcrush and Jpegtran.