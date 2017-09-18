Scalable storage of photos

In many Web applications, there is a need to have a system for storing, resizing and rendering photos. For example, for the function of uploading photos to a profile by users.

Simply storing and uploading downloaded images into the Web folder can be a big problem when it comes to thousands of files.

Resize pictures to prepare small thumbnails (thumbnails) is also nontrivial. Is it worth it to resize the pictures "on the fly" or immediately after downloading?

Where to store pictures, how best to give, how and how to resize, so that everything was quick and easy?

Uploading photos

Use a separate server to upload photos. In the future, such a solution is more convenient to scale. However, at the startup, the same main server application can perform the same functions: 

Loading of pictures will occur in two stages. First the pictures are uploaded to the main application server, and from there to the image server:

<?

# сюда загружается файл на основном сервере
$file = $_FILES['file']['tmp_name'];

$url = 'http://image.server.com/upload.php';
$post = [ 'file'=> new CURLFile($file, mime_content_type($file), 'file') ];

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result = curl_exec($ch);
# The script upload.php will receive and save the picture

Storage

Modern cloud services like Amazon S3 provide very favorable conditions for storing files. For 1TB of data you will pay about $ 30 a month. This is much more profitable than installing separate servers for storage.

Thus, our image server should save the picture in S3. For convenient operation, you can use S3-php :

<?


# загруженная фотка из приложения
$file = $file = $_FILES['file']['tmp_name'];
$key = md5($file) . '.jpg';


# сохраняем файл в облако
require 's3.php';
S3::$useExceptions = true;
$s3 = new S3('ключ', 'секрет');
$pubbed = $s3->putObjectFile($file, 'букет', $key, S3::ACL_PUBLIC_READ, []);
# The key, secret and bouquet should be obtained in the AWS console .

Return

In the simplest case, you can upload photos directly from S3. However, usually cloud services rate traffic and HTTP requests, so it is desirable to minimize them. For this, it makes sense to deploy a caching server (for example, Varnish ): 

Then the image server will perform two functions:

Uploading and copying files to the cloud.
Return files and cache them to optimize traffic.
In Varnish you need to configure the corresponding backend for servicing pictures from S3 (file default.vcl):

backend default {
  set backend.host = "букет.s3.amazonaws.com";
  set backend.port = "80";
}
  
sub vcl_recv {
  if (req.url ~ "\.(css|gif|ico|jpg|jpeg|js|png|swf|txt)$") {
    set req.backend = s3;
    lookup;
  }
}
# The bouquet needs to be used the same as during loading on S3

Resize pictures

Most of the pictures must be pre-processed before giving. For example, resize (resize) and crop to prepare smaller versions (thumbnails).

To process photos there are mega cool tools, for example, GraphicsMagick or ImagemMagick .

Processing photos is convenient to do during the recoil. This will avoid mass processing when changing and adding new dimensions. To get a reduced version of 100x100 for the photo, we will transfer the required size in the parameters:

http://image.server.com/?image=photo_key.jpg&size=100x100
The request will take over Varnish and transfer it to Nginx: 

We will use PHP as a tool for manipulating images:

<?

$image = $_GET['image'];
$local = '/tmp/' . md5($image);


# копируем файл с s3 на сервер
$s3_image = http://букет.s3.amazonaws.com/' . $image;
copy($s3_image, $local);


# небольшая валидация
$size = $_GET['size'];
if ( $size != '100x100' ) return;


# ресайзим и возвращаем в stdout картинку
$cmd = 'gm convert ' . $img . ' -resize ' . $size . ' - ';
header('Content-type: image/jpeg');
passthru($cmd);


# удаляем темповый файл
unlink($local);
Scaling

The resulting architecture is scaled linearly. You just need to increase the number of image servers: 

To ensure that there are no duplicate files in each cached server, you should divide the load and payback into subdomains:

i1.image.com
i2.image.com
...
New photos are uploaded to a random server. After that, we save not only the key of the picture, but also the server from which it is necessary to give it. When giving, we generate the final path:

http://i3.image.com/key.jpg
If one server goes down - just switch the return on another. Because, all the files in the cloud, nothing more will be needed.

The most important

Using cloud storage will significantly reduce the cost of administration and accessibility. Using cloud servers as nodes for caching will quickly add or remove new image servers depending on the load.

Such architecture is used in practice by large children, who serve hundreds of thousands of downloads and give tens of millions of photos per day. This scheme is implemented in the cloud service i .