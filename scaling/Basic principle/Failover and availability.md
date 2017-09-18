Failover and availability

The availability of any application depends on the health of its components. Ensuring the availability of the application is to ensure the availability of components. At the physical level, this reduces the dependency of the application's performance on server failures.

Sooner or later the server will break, no matter how reliable it is. The risk of failure of iron is solved only through redundancy . Those. when each node has a backup. And in case of the failure of their main server, the backup server can take over the work of the main server. In another way, this is called failover .

Redundancy

The main principle of fault tolerance is to avoid the number 1 ( SPOF ). Any component must be reserved. This is the principle of redundancy.

Backup components must repeat the configuration of the main (RAM, processor power, etc.). Two servers are weaker than one; The probability of failure of two servers is simultaneously much lower than one.

There are several simple approaches for reserving the main components of a Web application:

DNS
Front-line
Bekends
Database
1. DNS

Usually for one domain several NS records are indicated:

ruhighload.com. 172800 IN NS ns1.dnsimple.com.
ruhighload.com. 172800 IN NS ns2.dnsimple.com.
In the event that one of the NS servers fails, the requests are processed by others. 

This makes it possible to provide a high level of availability for servicing DNS queries. Make sure that your domain has several NS records:

dig ns ruhighload.com
2. Frontlines

A front-end is a server that receives requests from clients and sends them a response. Failure of such a server results in inaccessibility of the entire Web application.

It is convenient to implement frontend redundancy using virtual IP addresses of UCARP. Then one of the two identical servers gets the virtual IP to which the domain is bound. If the current server fails, this IP address is assigned to the backup server. 

Tracking the state and switching IP addresses should be automated. You can poll the main server every second for correct operation (the so-called heartbeat). In case the correct answer was not received, reassign the IP address to the backup frontend:

<?
while (true)
{
	# IP адрес фронтенда
	$c = curl_init('http://10.10.10.1/heartbeat.html');
	curl_setopt(CURLOPT_RETURNTRANSFER, true);
	$html = curl_exec($c);

	# что-то пошло не так, нужно менять IP
	if ( $html != 'ok' )
	{
		# Назначение виртуального IP второму фронтенду
		exec('kill ucarp');
		sleep(1);
	}
}
# Check heartbeat.html (contains the word "ok") every second

The lower level of monitoring (server availability) UCARP implements independently.

3. Bekends

Backends are the servers on which the main application runs (for example, PHP). They are usually absolutely identical. This allows you to use active redundancy. Those. all backend processes requests, there are no backup servers. If one of the backends fails, it is simply excluded from the general list, and requests from the frontend stop coming to him.

However, if you disconnect one of the servers, the load on the others will increase. The number of backends is better calculated on the basis that 25% of their total number can fail at any moment. 

Nginx allows you to specify multiple servers for processing fastcgi queries using upstream:

upstream backend {
        server 10.10.0.5;
        server 10.10.0.6;
        server 10.10.0.7;
}

server {
	location ~* \.(php)$ {
        fastcgi_pass backend;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
Requests will be automatically distributed between the specified servers.

In addition, Ngxinx will in real time analyze errors in replies from backends. If an error is detected, it will stop sending requests for the broken backend. The time and number of attempts can be adjusted:

upstream backend {
        server 10.10.0.5 fail_timeout=360s max_fails=2;
        server 10.10.0.6 fail_timeout=360s max_fails=2;
        server 10.10.0.7 fail_timeout=360s max_fails=2;
    }
# in case of 2 unsuccessful requests, Nginx will exclude the server from the pool for 360 seconds

max_fails determines the number of errors received from the backend, and fail_timeout is the time at which the backend will be excluded from the pool.

4. Databases

Database backup is the most difficult task. To ensure fault tolerance, Master-Slave replication is often used . It should be noted that the replica in this case works in the passive mode. Those. It does not process any requests from the application, it only serves to store the current copy of the data from the wizard.

It is convenient to automate the tracking of the state of databases. If an error is detected, the settings in the application are automatically changed to the replica server:

<?
$master = include 'config/db/master.php';
$master_replica = include 'config/db/replica.php';

# Проверяем выполнение запроса на мастере
mysql_connect($master);
mysql_query('SELECT NOW()');

# Произошла ошибка
if ( mysql_errno() )
{
	$config = "<? return '{$master_replica}';";
	file_put_contents('config/db/master.php', $config);
}
# We overwrite the settings file with the address of the Wizard in case of an error

The process of repairing a broken server will be as follows:

A broken server is disconnected from any operations and is repaired.
After the restore, it configures replication from the running server.
After configuring the replication and synchronizing it, the configuration of this server is added to the application (in our example we would add its address to the file " config / db / replica.php ").
The slave can be configured from the working wizard using the Xtrabackup utility .

After that, the former slave becomes the main (ie, the Wizard), and the restored server becomes a backup (ie Slave).

The most important

Any server will sometime break down. Avoid nodes that work in a single instance. Give preference to less powerful servers, but more of them.

Remember, the main method of providing fault tolerance is redundancy. When planning the expansion, consider the need to purchase backup equipment.