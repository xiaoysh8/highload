Amazon S3: Store Overview

Amazon Simple Storage Service (S3) - storage of files of any type, any volume, with high availability and fault tolerance. It is designed to store static content, user data and backups.Amazon S3 simple scheme

The service is suitable for small, young projects with several thousand files and relatively small traffic, because the cost will be as low as possible (about several dollars a month).

But as the cost increases proportionally, you need to pay not only for downloading and distributing files, but also for traffic, GET and POST requests.

Advantages and disadvantages

The main disadvantage of S3 is the high cost when loading and retrieving terabytes of data.

But the advantages are obvious:

Reliability and security - distributed data backups (automatically), support for SSL, encryption, access permissions;
High availability - several regions to increase download speed, guarantee the availability of data;
Simplicity of scaling - automatic increase in volume as required;
Withstands any load - sharp peaks of popularity will not affect the availability of files;
Easy to use - web console, API, SDK and even mobile application.
Capabilities

The whole cycle of work with the service is limited to 5 actions: S3 cycle

All files (photos, videos, documents, etc.) are stored in containers ( bucket ). The easiest way to create new buckets is to use the Amazon S3 Web Console . And this is implemented by means of PHP:

$result = $client->createBucket([
    'ACL' => 'private',
    'Bucket' => 'test-bucket',
    'CreateBucketConfiguration' => [
    'LocationConstraint' => 'eu-central-1',
    ],
]);
# With access rights, container name and region

You can also use the SDK for Ruby, Java and .NET ( quite complex and extensive ), and curl in the form:

curl -v https://testbucket.s3.amazonaws.com/ \
     -H "Authorization: AWS4-HMAC-SHA256 \
         Credential=AKIAIOSFODNN7EXAMPLE/20160530/us-east-1/s3/aws4_request, \
         SignedHeaders=host;x-amz-content-sha256;x-amz-date, \
         Signature=182072eb53d85c36b2d791a1fa46a12d23454ec1e921b02075c23aee40166d5a" \
     -H "x-amz-content-sha256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855" \
     -H "x-amz-date: 20160530T124500Z"
# Create a new container with credentials

The name of the container can be represented as a URL:

http://bucket.s3-aws-region.amazonaws.com


# Или

http://s3-aws-region.amazonaws.com/bucket
# Brief region names are available in the documentation, by default US East (N. Virginia)

Command Line Utilities

To work with S3 there is a sufficient number of 3rd party tools, one of the best - console s3cmd for Ubuntu:

sudo aptitude install s3cmd
# The utility is available in the aptitude system

Before using s3cmd , you need to edit the configuration file ~ / .s3cfg with the S3 credentials. Then the utility will provide simplified storage management:

s3cmd mb s3://newbucket							# создать контейнер newbucket

s3cmd rb s3://newbucket							# удалить пустой контейнер newbucket

s3cmd ls [s3://BUCKET[/PREFIX]]					# список файлов или корзин

s3cmd la										# список всех объектов во всех корзинах

s3cmd put FILE [FILE...] s3://BUCKET[/PREFIX]	# загрузить файл в корзину

s3cmd get s3://BUCKET/OBJECT LOCAL_FILE			# скачать файл

s3cmd del s3://BUCKET/OBJECT					# удалить файл

s3cmd sync LOCAL_DIR s3://BUCKET[/PREFIX]		# синхронизировать каталог S3 с локальным каталогом
s3cmd sync s3://BUCKET[/PREFIX] LOCAL_DIR		# синхронизировать каталог S3 с локальным каталогом

s3cmd du [s3://BUCKET[/PREFIX]]					# показать объем файлов

s3cmd setacl s3://BUCKET[/OBJECT]				# изменить параметры доступа к объекту

s3cmd info s3://BUCKET[/OBJECT]				  # свойства объекта
# All commands are in the help s3cmd --help

A similar solution is provided by Amazon itself. But AWS CLI is more versatile and works with all cloud services of the company:

curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
$ unzip awscli-bundle.zip
$ sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
# Download the package from the S3 recycle bin, unpack and install, you need installed Python

After installation, you also need to configure your profile:

$ /usr/local/bin/aws configure --profile test-bucket
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: aws-region
Default output format [None]: json
# Enter your credentials, if you do not specify a profile, the primary

The utility contains a large number of options and additional functions .

The most important thing

Start acquaintance with Amazon S3 through the web console - it will give a visual representation of your repository. In most cases, it will be enough to work with the service. For advanced work with the repository, SDK and other utilities are available .