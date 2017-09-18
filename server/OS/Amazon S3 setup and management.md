Amazon S3: setup and management

The main scenarios for working with S3 are backups and static files . And, automation of actions is welcomed.Amazon S3 usage

Syncing files

To upload files to S3, synchronize and manage objects, any existing clients, web console, plug-ins, command-line utilities and REST APIs are used. In the simplest case, it is enough to have AWS CLI for managing baskets and objects:

$ aws s3 sync s3://test-bucket /usr/local/test-bucket/ --metadata-directive REPLACE --expires "Wed, 7 Jun 2017 08:16:32 GMT" --cache-control "max-age=2592000"
# File to synchronize with the repository, caching and age

Another method is a simple bash script :

file=/path/to/file/to/upload
bucket=your-bucket
resource="/${bucket}/${file}"
contentType="application/x-compressed-tar"
dateValue=`date -R`
stringToSign="PUT\n\n${contentType}\n${dateValue}\n${resource}"
s3Key=xxxxxxxxxxxxxxxxxxxx
s3Secret=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
signature=`echo -en ${stringToSign} | openssl sha1 -hmac ${s3Secret} -binary | base64`
curl -X PUT -T "${file}" \
  -H "Host: ${bucket}.aws-region.amazonaws.com" \
  -H "Date: ${dateValue}" \
  -H "Content-Type: ${contentType}" \
  -H "Authorization: AWS ${s3Key}:${signature}" \
  https://${bucket}.aws-region.amazonaws.com/${file}
# Be sure to specify the path to the file, the trash folder name and the AWS region

This example can easily be turned into a bash script that will create backups of the necessary files and upload them to S3:

#!/bin/bash

cd /tmp
rm -rf backup
mkdir backup
cd backup

mkdir nginx && cd nginx
cp -R /etc/nginx/sites-enabled .
cp /etc/nginx/nginx.conf .

cd ..
mkdir git && cd git
repos=`ls -1 /home/git | grep '.git$'`
for repo in $repos; do
    cp -R "/home/git/${repo}" .
done    

cd ..
date=`date +%Y%m%d`
bucket=my-bucket
for dir in git nginx; do
    file="${date}-${dir}.tar.gz"
    cd $dir && tar czf $file *
    resource="/${bucket}/${file}"
    contentType="application/x-compressed-tar"
    dateValue=`date -R`
    stringToSign="PUT\n\n${contentType}\n${dateValue}\n${resource}"
    s3Key=xxxxxxxxxxxxxxxxxxxx
    s3Secret=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    signature=`echo -en ${stringToSign} | openssl sha1 -hmac ${s3Secret} -binary | base64`
    curl -X PUT -T "${file}" \
        -H "Host: ${bucket}.s3.amazonaws.com" \
        -H "Date: ${dateValue}" \
        -H "Content-Type: ${contentType}" \
        -H "Authorization: AWS ${s3Key}:${signature}" \
        https://${bucket}.s3.amazonaws.com/${file}
    cd ..
done

cd
rm -rf /tmp/backup
# Uploading backup copies of the Nginx configuration file and the Git repository

PHP SDK

To use PHP for backup, you need to install its SDK for Amazon AWS. It will require PHP and Composer installed :

curl -sS https://getcomposer.org/installer | php # загрузка с офф. страницы


# установка последней версии SDK
php composer.phar require aws/aws-sdk-php
# Composer is recommended, but not required

And then you need to enable the autoloader:

<?php
require 'vendor/autoload.php';
# Do not forget to add to your scripts

Optionally you can install via Phar or ZIP-file :

		# для Phar
<?php
require '/path/to/aws.phar';
	# для zip
require '/path/to/aws-autoloader.php';
# It is necessary to unpack files and include them in scripts

First you need to configure the client for S3:

<?php

# Включить SDK при помощи Composer
require 'vendor/autoload.php';

$s3 = new Aws\S3\S3Client([
    'version' => 'latest',
    'region'  => 'us-east-1'
]);
# It is better to enter the data via the environment variable or ini-file in the AWS directory

The SDK allows you to use classes to apply common configurations between different clients:

# Указание региона и последней версии клиентов
$sharedConfig = [
    'region'  => 'us-west-2',
    'version' => 'latest'
];

# Создать класс SDK
$sdk = new Aws\Sdk($sharedConfig);


# Создать клиент Amazon S3 с общими конфигурациями
$client = $sdk->createS3();
# The options common to all clients are placed at the root in the form of a key-value

To perform operations, you must call the method in the client's name:

 # Используется класс Aws\Sdk 
$s3Client = $sdk->createS3();


# Отправка запроса PutObject и получение результата
$result = $s3Client->putObject([
    'Bucket' => 'my-bucket',
    'Key'    => 'my-key',
    'Body'   => 'this is the body!'
]);


# Загрузка объекта
$result = $s3Client->getObject([
    'Bucket' => 'my-bucket',
    'Key'    => 'my-key'
]);


# Выводит тело результата
echo $result['Body'];
# SDK uses a configuration file that matches the required version

Removing the container in its simplest form looks like this:

$result = $client->deleteBucket([
    'Bucket' => 'test-bucket',
]);
# Container must be empty

A deletion of objects occurs as follows:

use Aws\S3\S3Client;

$s3 = S3Client::factory();

$bucket = 'test-bucket';
$keyname = 'object-key';

$result = $s3->deleteObject(array(
    'Bucket' => $bucket,
    'Key'    => $keyname
)); 
# Do not forget to specify the object key

In addition, the system supports asynchronous and HTTP-requests.

Given the wide capabilities of the SDK, uploading backups to S3 is as simple as possible:

putenv('AWS_ACCESS_KEY_ID=Your_key_ID');
putenv('AWS_SECRET_ACCESS_KEY=Your_access_ID');
putenv('S3_BUCKET=bucket-name');
$s3 = new Aws\S3\S3Client(['version' => '2006-03-01', 'region' => 'eu-central-1', 'signature_version' => 'v4']);
$s3->upload('backup', 'path' . date('Y_m_d') . '.gz', fopen($dump, 'rb'));
# Specify authorization data via environment, version, and region variables

The life cycle of objects

To automatically delete objects of a certain age, it's easiest to use the Amazon AWS web console, specifying expiration for the files in it . If you need automation, then you can use the scripts SDK or S3 console, which will run periodically .

And there are also life cycle policies ( lifecycle policy ). These are forms (in XML), a set of rules consisting of rule ID, status (on / off), types of objects that are covered by the rule, carry and expiration of objects.

It looks like this:

<LifecycleConfiguration>
    <Rule>
        <ID>sample-rule</ID>
        <Prefix></Prefix>
        <Status>Enabled</Status>
        <Transition>Disabled</Transition>    
        <Expiration>
             <Days>365</Days>
        </Expiration>
    </Rule>
</LifecycleConfiguration>
# All objects older than 365 days will be deleted

This set of rules applies to the correct container (written to container properties):

$ aws s3api put-bucket-lifecycle  \
--bucket bucketname  \
--lifecycle-configuration filename-containing-lifecycle-configuration
# You must specify the name of the container and the name of the XML file that is in the local directory

Simple policies for moving and deleting files are conveniently created using the AWS web console .

You can also put the rules in the JSON file:

{
    "Rules": [
        {
            "Status": "Enabled",
            "Prefix": "logs/",
            "Expiration": {
                "ExpiredObjectDeleteMarker": true
            },
            "ID": "TestOnly"
        }
    ]
}
# Will be applied to all log files

After that, you can automatically apply the necessary rules to new objects using a simple script that will contain the command:

$ aws s3api put-bucket-lifecycle  \
--bucket bucketname  \
--lifecycle-configuration file://lifecycle.json
# The same principle checks and removes rules from containers

S3 and Nginx

Nginx can distribute static files that are stored on S3. Amazon S3 and Nginx

You need to edit its configuration file by specifying S3 in the location section :

location / {
  set $s3_bucket        “BUCKET.s3.amazonaws.com';
  set $aws_access_key   'AWSAccessKeyId=ACCESS_KEY';
  set $url_expires      'Expires=$arg_e';
  set $url_signature    'Signature=$arg_st';
  set $url_full         '$1?$aws_access_key&$url_expires&$url_signature';
  proxy_http_version    1.1;
  proxy_set_header       Host $s3_bucket;
  proxy_set_header       Authorization '';
  proxy_hide_header      x-amz-id-2;
  proxy_hide_header      x-amz-request-id;
  proxy_hide_header      Set-Cookie;
  proxy_ignore_headers   "Set-Cookie";
  proxy_buffering        off;
  proxy_intercept_errors on;
  resolver              172.16.0.23 valid=300s;
  resolver_timeout      10s;
  proxy_pass             http://$s3_bucket/$url_full;
}
# Do not forget to specify your containers and access keys

The most important thing

Automate all the tasks of downloading and managing the Amazon S3 . The presence of SDK, console utility and API will allow to use all the capabilities of the storage.