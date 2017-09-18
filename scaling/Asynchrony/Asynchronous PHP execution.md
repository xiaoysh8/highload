Asynchronous PHP execution

To accelerate the work of the program, the practice of asynchronous task execution is widely used . This means that the operations are performed inconsistently, deferred.

The choice of an asynchronous solution is always non-trivial, especially if you run PHP.

PHP-FPM fastcgi_finish_request ()

If Nginx is used as the web server , then the excellent FastCGI process manager (FPM) is at your service . fastcgi_finish_request

For example, sending mail in asynchronous mode will look like this:

<?
$to = $_POST['email'];

if ( $to )
{
	echo 'Подтверждение отправлено на почту';
	fastcgi_finish_request();
	mail($to, 'Подтверждение', 'Это ваш ящик?');
}
else
{
	echo 'Введены не все данные';
}
# After calling fastcgi_finish_request (), the visitor will immediately see the response from the server

More options and examples in the material Asynchrony in PHP and FPM .

Non-blocking flow mode

By default, the socket is opened in blocking mode, that is, it waits for a complete connection before returning a value. So to perform asynchronous calls, the additional socket_set_nonblock directive is useful :

<?php 

function multiHTTP ($urlArr) { 
$sockets = Array(); // массив сокетов
$urlInfo = Array(); 
$retDone = Array(); 
$retData = Array(); 
$errno   = Array(); 
$errstr  = Array(); 
for ($x=0;$x<count($urlArr);$x++) { 
  $urlInfo[$x] = parse_url($urlArr[$x]); 
  $urlInfo[$x][port] = ($urlInfo[$x][port]) ? $urlInfo[$x][port] : 80; 
  $urlInfo[$x][path] = ($urlInfo[$x][path]) ? $urlInfo[$x][path] : "/"; 
  $sockets[$x] = fsockopen($urlInfo[$x][host], $urlInfo[$x][port], 
                           $errno[$x], $errstr[$x], 30); 
  socket_set_nonblock($sockets[$x]); 
  $query = ($urlInfo[$x][query]) ? "?" . $urlInfo[$x][query] : ""; 
  fputs($sockets[$x],"GET " . $urlInfo[$x][path] . "$query HTTP/1.0\r\nHost: " . 
        $urlInfo[$x][host] . "\r\n\r\n"); 
} 
// чтение данных
$done = false; 
while (!$done) { 
  for ($x=0; $x < count($urlArr);$x++) { 
   if (!feof($sockets[$x])) { 
    if ($retData[$x]) { 
     $retData[$x] .= fgets($sockets[$x],128); 
    } else { 
     $retData[$x] = fgets($sockets[$x],128); 
    } 
   } else { 
    $retDone[$x] = 1; 
   } 
  } 
  $done = (array_sum($retDone) == count($urlArr)); 
} 
return $retData; 
} 
?>
# The function is connected to an array of URLs in asynchronous mode and returns an array of results

You can also enable non-blocking flow mode. To do this, use the stream_set_blocking function . For example, in a non-blocking mode, the call to fgets () will return immediately, without waiting for the availability of all data on the thread.

A small example of use:

function call_url($url) {
    
    $fp = fopen($url, 'r');
    stream_set_blocking($fp, 0);
    $data = fread($fp, 8192);
    fclose($fp);
     
    return strlen($data);
}
# Call url and instant response, convenient for running other scripts

Exec and cURL

To start background processes curl , you can use the exec. A simple implementation example would look like this:

<?php
  $ch = curl_init();
 
curl_setopt($ch, CURLOPT_URL, 'http://somesite.com/long-script.php');
curl_setopt($ch, CURLOPT_FRESH_CONNECT, true);
curl_setopt($ch, CURLOPT_TIMEOUT_MS, 50);
 
curl_exec($ch);
curl_close($ch);
?>
# The query is executed almost instantly (50 ms), and the required long script is run in the background

Curl processes can also be run from under the shell:

curl -X POST -H 'Content-Type: application/json' \
  -d '{"batch":[{"userId":"some_user","event":"PHP Fork Event","timestamp":"2016-05-16T14:34:50-08:00","context":{"library":"analytics-php"},"action":"track"}]}' \
  'https://api.somesite.com/import' > /dev/null 2>&1 &
# Run and split the script

Expansion of pthreads

For PHP, there is a multi-threaded extension for pthreads . php pthreads

It is not part of the PHP kernel, so you need to install it:

pecl install pthreads
# The extension is available in the PECL repository and is compatible with ZTS

The simplest example to test the performance of an extension would be:

<?php

class workerThread extends Thread {
public function __construct($i){
  $this->i=$i;
}

public function run(){
  while(true){
   echo "Worker {$this->i} ran" . PHP_EOL;
   sleep(1);
  }
}
}

for($i=0;$i<7;$i++){
$workers[$i]=new workerThread($i);
$workers[$i]->start();
}

?>
# Display of numbers of vorkers in 8 streams

Using the Queue Server

If the web application is large, then it is more logical to use the queue server, which will give more features and functions. Gearman PHP queue

For example, Gearman , which contains many clients, including PHP. With its help it is quite easy to implement asynchronous operations, like sending mail .

The most important thing

The choice of the asynchronous execution method depends entirely on your tasks. But the most productive and reliable solutions will be using the PHP-FPM module and the queue server.