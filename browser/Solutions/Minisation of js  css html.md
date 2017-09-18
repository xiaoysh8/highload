Minisation of js / css / html

Minify is a simple approach for reducing the size of css, js and html files. During compression, all comments to the code, line breaks, extra tabs and whitespace are deleted. This allows you to save 10 ... 20% of the original file size. Minify css

CSS and Javascript

There is a whole bunch of tools that solve this problem for CSS and JS files. For example, YUI compressor from Yahoo. You simply run a utility that will save the processed copy to a file:

java -jar yuicompressor-xyzjar myfile.js -o myfile-min.js
HTML

With HTML a little more difficult, because This task will have to be performed in the application itself. This will make sense if the size of your HTML files exceeds 10 KB. Then the savings can be palpable. In PHP, this might look like this:

<?php
function sanitize_output($buffer) {
    $search = array(
        '/\>[^\S ]+/s',  
        '/[^\S ]+\</s', 
        '/(\s)+/s'
    );
    $replace = array(
        '>',
        '<',
        '\\1'
    );
    $buffer = preg_replace($search, $replace, $buffer);
    return $buffer;
}
ob_start("sanitize_output");
?>
<html>
<body>
...
# The sanitize function removes unprintable characters from HTML before it is output

For automation of work it is necessary to use PHP Minify .

How it is done in practice

You need to prepare a mini-copy of all your CSS / Javascript / HTML files. Usually this is done during the rollout phase of the new version:

Retrieving all required files
Minimization, for example with the help of YUI compressor
Roll-out to the product
Make sure that the requests from the product will be received on the minified files:

<script src="/min.jquery.js"></script>
PHP script that will find all the JS / CSS files in the folder and minify them with the name "min." + filename:

<?
$path = '.';
$yui_path = 'yui.jar';
$list = array();
function minify_project($dir)
{
	$files = glob( $dir, GLOB_NOSORT );
	foreach ( $files as $file ) if ( in_array( pathinfo( $file, PATHINFO_EXTENSION ), array('css', 'js') ) )
	{
		echo $file . "\n";
		exec( 'java -jar ' . $yui_path . '  --charset utf-8 ' . $file . ' -o ' . (dirname($file) . '/min.' . basename($file)) );
	}
	else if ( is_dir($file) )
	{
		minify_project( $file . '/*' );
	}
}

minify_project($path . '/*');
# Specify $ path to the project folder and $ yui_path to the yui script

The most important

Be sure to minify JS and CSS, because this simple operation saves 10 ... 20% of the data volume. Minify HTML only if it is large enough and takes at least 10% of the size of the request.