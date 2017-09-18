Amazon S3 Prices: Methods of Economy

Amazon S3 provides an almost unlimited amount of storage, which will cost quite a bit for a small web project. But with the increase in the amount of data, you need to pay not only for the storage, but also for traffic, requests GET, PUT, COPY, POST and LIST.Amazon S3 pricey

Cost

Prices are listed on the AWS page , you can count in the calculator .

Free limited access gives 5 GB of storage, 20,000 GET requests and 2000 PUT requests per year. Then you'll have to shell out $ 30 for a terabyte of storage and $ 90 for each terabyte of traffic. A site with 100 thousand views a day, which contains 5 photos on each page, will generate such traffic. And with the growth of popularity in 10 times pay will almost $ 1000 per month.

Cloudfront

Amazon S3 is not a CDN . It is much cheaper to distribute statics for a popular web application using your own servers or a separate CDN provider.

Amazon has its own content delivery network - CloudFront . It is faster and cheaper than S3, and is also integrated into other AWS services.

Data transfer from S3 to CloudFront is free, downloading data to CDN from an external source costs only $ 20 for 1 TB, and the distribution of the first 10 TB per month will cost $ 850. At the same time, the price decreases with increasing traffic.

The easiest way to connect CloudFront via the AWS web console, or using the CLI:

aws cloudfront create-distribution \
  --origin-domain-name my-bucket.s3.amazonaws.com \
  --default-root-object index.html
# Specifies the domain or container S3

Caching

The amount of traffic most significantly affects the cost of S3. Amazon S3 caching

Therefore, caching will reduce traffic and a monthly check from Amazon - the files are downloaded to S3, and are returned through their own cache servers.

For this, Varnish is suitable , which can work with AWS:

backend s3 {
  .host = "s3.amazonaws.com";
  .port = "80";
}

sub vcl_recv {
  if (req.url ~ "\.(gif|jpg|jpeg|png)$") {
      unset req.http.cookie;
      unset req.http.cache-control;
      unset req.http.pragma;
      unset req.http.expires;
      unset req.http.etag;

      set req.backend = s3;
      set req.http.host = "test-bucket.s3.amazonaws.com";

      lookup;
  }
}

sub vcl_fetch {
    set obj.ttl = 3w;
}
# Will cache pictures from test-bucket

BitTorrent

If you plan to upload large files via S3, it makes sense to use the built-in BitTorrent support. So using the peer-to-peer network, each client will download and distribute files, thereby reducing S3 traffic.

Create a torrent file is very simple - you need to add a torrent to the link to the file hosted on S3:

 # Было
http://s3.amazonaws.com/test_bucket/somefile


# Стало
http://s3.amazonaws.com/test_bucket/somefile?torrent
# Clients automatically download the torrent file

Glacier

If S3 is mostly used to store backups, consider using Glacier, a backup storage. Service in any case will not replace S3, but will allow you to keep rarely used files that are not needed in real time.

It is noteworthy that Glacier, because of its limitations, is significantly cheaper than S3 - storing 1 TB of data will cost $ 7 per month, and downloading 1 TB in $ 90.

To connect the service, you need to create a life-cycle policy by including Glacier in transfer :

<LifecycleConfiguration>
    <Rule>
        <ID>sample-rule</ID>
        <Prefix></Prefix>
        <Status>Enabled</Status>
        <Transition>
      		<Days>30</Days>
      		<StorageClass>GLACIER</StorageClass>
    	</Transition>    
        <Expiration>
             <Days>365</Days>
        </Expiration>
    </Rule>
</LifecycleConfiguration>
# Automatically upload files to Glacier after 30 days

The most important thing

Do not use the Amazon S3 as a CDN, caching with some low-cost servers and turning on BitTorrent for uploading files can save a lot without degrading the performance of your web application.