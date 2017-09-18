High Load Architecture

Architectural solutions are the foundation for building any applications. Including applications with high loads. It is important to understand that the architecture of a web application determines 95% of the success of its work. Including the ability to cope with the load.

Principles of development

Developing a successful, and therefore a great, web application, you need to understand the principles of building large systems.

Dynamics

You never know what will happen to the application tomorrow. Perhaps, the number of your users will increase 5 times. Perhaps, a minor function will start to gain popularity sharply. And it will create absolutely new problems. The more the system becomes, the less effective will be the long-term planning.

The success of work on a large resource implies not a detailed planning of all aspects. The main effort should be to ensure the flexibility of the system. Flexibility will allow you to make changes quickly. This is the most important property of any fast-growing system.

Gradual growth

Do not try to predict the audience size for a year ahead. The same goes for the application architecture. The basis for successful development is gradual solutions. This applies to both the software and the hardware.

If you are running a new application, it makes no sense to immediately provide an infrastructure that can withstand millions of visitors. Use Cloud solutions to host new projects. This will reduce server costs and simplify management.

Many Cloud Hosting services provide private network services. This will allow you to use multiple servers in the cloud together. This way you can perform the first steps in scaling right in the cloud.

Simple solutions

Simple solutions are extremely difficult to develop. However, it is better to spend time and effort on simplifying solutions (both for development and for users). A flexible system is not difficult.

Progressive changes

Work on a large project is very similar to downloading a picture of the Progressive JPEG format . You move not gradually, but chaotically. You will have to constantly modify and modify various solutions, switch from one to another.

95% percentile

95% percentile
Apply the 95% percentile rule. You should only spend 95% of your time. The remaining 5% are usually special cases, which lead to an extremely complicated system. For example:

Some users in social. networks can have tens of thousands of links. If they are less than 5%, put a restriction and do not solve this problem while there are more important problems.
Some users load videos that have non-standard encoding. Show the error instead of wasting time on converting the converter.
Less than 5% of users use browsers with cookies disabled, Javascript restrictions, etc. Disable the ability to view the site for them and post detailed instructions on updating the browser instead of adapting the site for them.
Focus on the important. Remember, you will always have more tasks than the time to solve them. Set priorities correctly - solve the problems that arise for the absolute majority of users.

Architectural solutions

Scaling of any Web application is a gradual process that includes:

Load analysis .
Determination of the most affected areas.
The transfer of such sites to individual nodes and their optimization.
Repeat step 1.
1. Simple web application architecture

A new application is usually run on a single server running both the Web server and the database and the application itself: Simple application architecture

This is reasonable, because this saves time and money on the launch. Use this approach to start. If you are afraid not to withstand the starting load - take a powerful server for rent. Only in exceptional situations, when you are absolutely sure of a large initial load - go directly to what is described below.

2. Branch of the database

Most often, the first node that is under load is the database. It's clear. Each request from the user is usually 10 to 100 database queries: Branch of the database

Putting a database on a separate server will increase its performance and reduce its negative impact on other components (PHP, Nginx, etc.). To connect to MySQL on a separate server, use the IP address of this server:

mysql_connect('10.10.0.2', 'user', 'pwd');
# 10.10.0.2 - IP address of MySQL on the internal network

Pause when migrating

Moving the database to another server can be a problem for the running application, because will take some time. You can:

Use a simple solution - post an announcement of scheduled work on the site and make a transfer. It is better to do this at night, when the activity of the audience is minimal.
Use replication to synchronize data from one server to another. In this case, the master is the old server, and the slave is the new one. After configuration it is enough to change the IP address of the database in the application to the new server. And then - turn off the old server. Read how to configure MySQL replication on a running server without downtime . .
Once MySQL has been allocated to a separate server, make sure that it is optimally configured in order to maximize the iron.

Scaling of databases is one of the most difficult tasks during the growth of the project. There are a lot of practices - denormalization, replication, sharding and many others. Read the materials on database scaling .

3. Separation of the Web server

Next on the queue is the Web server. Its allocation to a separate node will leave more resources for the application (in the example, PHP): Separation of the Web server

In this case, you will need to configure the deployment of the application to both the Nginx server and the server with PHP. A PHP server is usually called a backend . Then Nginx will give the static files themselves, and the PHP server will only be occupied with processing the scripts. Nginx allows you to connect to the backend by IP address:

server {
	server_name ruhighload.com;

	root /var/www/ruhighload;
	index index.php;

	location ~* \.(php)$ {
        fastcgi_pass 10.10.10.1:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
# All files of statics Nginx will give without reference to the backend

If you use file download, you will need to allocate the file storage to a separate node (see below about this).

4. Several PHP backends

When the load grows, the Web application gradually starts to work more slowly. At some point, the reason will lie in the very implementation of the application. Then it's worth installing several PHP backends: Multiple PHP backends

All backends are desirable to have the same configuration. Nginx can balance the load between them. To do this, you need to select the list of backends in upstream and use it in the configuration:

upstream backend {
    server 10.10.10.1;
    server 10.10.10.2;
    server 10.10.10.3;
}

server {
	server_name ruhighload.com;

	root /var/www/ruhighload;
	index index.php;

	location ~* \.(php)$ {
        fastcgi_pass backend;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
# Nginx will evenly distribute the load between the specified backends

You can use weight backends, if any of them are more powerful than others:

upstream backend {
    server 10.10.10.1 weight=10;
    server 10.10.10.2 weight=2;
    server 10.10.10.3 weight=4;
}
# For every 16 queries, the first backend will process 10, the second backend will process 2, and the third - 4

Sessions

Once you start using multiple backends, requests from the same user will be sent to different servers. This will require a single repository for sessions, for example Memcache.

5. Caching

Connecting cache servers is one of the simplest tasks: Architecture - caching

Memcache provides the use of multiple servers in a standard library:

<?
$m = new Memcache;
$m->addServer('10.5.0.1');
$m->addServer('10.5.0.2');
...
$m->get('user1')
# Connect Memcache to multiple servers at once

Memcache will independently distribute the load between the servers used. To do this, he uses a constant hashing algorithm . You will need to monitor the repression and add new equipment on time.

6. Task queues

Task queues allow you to perform heavy operations asynchronously without slowing down the main application. Architecturally it looks like this:Task queue architecture

The queue server receives tasks from the application. The server's worker processes the tasks. Their number should be increased when the average number of tasks in the queue will gradually increase.

7. DNS Balancing

DNS supports balancing based on Round Robin . This allows you to specify several IP addresses of the receiving Web servers (usually called frontends):DNS round robin

To use this mechanism, you need to install several identical frontends. Then in the DNS should specify such A records:


....
ruhighload.com    IN  A       188.226.228.90
                  IN  A       188.226.228.9x
                  IN  A       188.226.228.9x
# We use several IP addresses for one A record

In this case, DNS will give different IP addresses to different clients. Thus, there will be a balancing between the front-ends.

8. File Storage

Uploading and processing files usually occurs on the backend. When there are several backends, it is inconvenient:

I'll have to remember which backend the file was uploaded to.
Downloading and processing files (for example, video or photo) can greatly reduce the performance of the backend.
You'll need to use a server with large disks, which is usually not necessary for backends.
The correct solution will be the use of separate servers for loading, storing and processing files: File Storage Architecture

In practice, this is implemented as follows:

A separate subdomain for the file server is selected.
Nginx is deployed on the server and a small application that can store (and process, if necessary) files.
Scaling occurs by adding new servers and subdomains (images1, images2, images3, etc.).
Uploading files

It is convenient to transfer the load to the client side. Then the form will send a request to a specific server:

<form action="http://images1.ruhighload.com/upload.php" method="post" enctype="multipart/form-data">
  <input type="file" name="file">
  <input type="submit" value="Загрузить">
</form>
Domains can be generated at random from existing ones:

<form action="http://images<?=mt_rand(1, 10)?>.ruhighload.com/upload.php" method="post" enctype="multipart/form-data">
...
AJAX download

To implement AJAX downloads, you will need to install HTTP headers for applications that accept files:

<?
header('Access-Control-Allow-Origin: http://ruhighload.com');
...
# This will send AJAX requests from the domain ruhighload.com to domains for downloading files

Read more about the structure of storing photographs and media data in large projects.

The most important

Remember - not with technology, but with architecture. It's not so important which technologies you use. It is much more important how you use them. Scale gradually and read the material on architectural solutions .