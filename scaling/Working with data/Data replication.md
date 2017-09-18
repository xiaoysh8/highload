Data replication

Replication is one of the techniques for scaling databases. This technique consists in the fact that data from one database server is constantly copied (replicated) to one or more others (called replicas). For the application, you can use more than one server to process all requests, but several. Thus, it becomes possible to distribute the load from one server to several.Database replication

There are two main approaches when working with data replication:

Master-Slave replication;
Master-Master replication.
Master-Slave replication

In this approach, one main database server is selected, which is called the Wizard. All changes to the data occur on it (any MySQL INSERT / UPDATE / DELETE requests). The slave server constantly copies all the changes from the wizard. From the application to the Slave server, data read requests (SELECT requests) are sent. Thus, the Master Server is responsible for changing the data, and the Slave for reading. 

Read how to configure Master-Slave replication to MySQL.

In the application, you need to use two connections - one for the Wizard, the second for Slave:

<?
$master = mysql_connect('10.10.0.1', 'root', 'pwd');
$slave = mysql_connect('10.10.0.2', 'root', 'pwd');

# ...
mysql_query('INSERT INTO users ...', $master);

# ...
$q = mysql_query('SELECT * FROM photos ...', $slave);
# We use two connections - for the Wizard and Slave - for writing and reading, respectively

Several Slaves

The advantage of this type of replication is that you can use more than one Slave. Usually, you should use no more than 20 Server Slaves when working with the same Wizard. 

Then from the application you select randomly one of the Slaves for processing requests:

<?
$master = mysql_connect('10.10.0.1', 'root', 'pwd');
$slaves = [
	'10.10.0.2',
	'10.10.0.3',
	'10.10.0.4',
];
$slave = mysql_connect($slaves[array_rand($slaves)], 'root', 'pwd');

# ...
mysql_query('INSERT INTO users ...', $master);

# ...
$q = mysql_query('SELECT * FROM photos ...', $slave);
Replication delay

Asynchronous replication means that the data on the Slave can appear with little delay. Therefore, in sequential operations it is necessary to use the read from the Wizard to get the actual data:

<?
$master = mysql_connect('10.10.0.1', 'root', 'pwd');
$slave = mysql_connect('10.10.0.2', 'root', 'pwd');

# ...
mysql_query('UPDATE users SET age = 25 WHERE id = 7', $master);
$q = mysql_query('SELECT * FROM users WHERE id = 7', $master);

# ...
$q = mysql_query('SELECT * FROM photos ...', $slave);
# When accessing the data to be changed, you must use the Master connection

Failure

If Slave fails, simply switch the entire application to work with the Wizard. After that, restore the replication to Slave and start it again.

If the Master fails, you need to switch all operations (and reads and writes) to the Slave. So he will become a new Master. After restoring the old Wizard, customize the replica on it, and it will become the new Slave.

Reservation

Much more often, Master-Slave replication is used not for scaling, but for redundancy. In this case, the Master Server processes all requests from the application. The slave server works in passive mode. But in the event of a failure of the Master, all operations are switched to the Slave.

Master-Master replication

In this schema, any of the servers can be used for both read and write: 

Read about the configuration of Master-Master replication in MySQL.

When using this type of replication, it is sufficient to select an accidental connection from the available Masters:

<?
$masters = [
	'10.10.0.1',
	'10.10.0.2',
	'10.10.0.3',
];
$master = mysql_connect($masters[array_rand($masters)], 'root', 'pwd');

# ...
mysql_query('INSERT INTO users ...', $master);
# Select a random wizard to process connections

Failure

Potential breakages make Master-Master replication unattractive. The failure of one of the servers almost always leads to the loss of some data. The subsequent recovery is also greatly hampered by the need for manual analysis of the data, which either did not have time to be copied.

Use Master-Master replication only as a last resort. Instead, it is better to use the technique of "manual" replication, described below.

Asynchronous replication

In MySQL, replication works in asynchronous mode. This means that the application does not know how fast the data will appear on the Slave. 

The delay in replication (replication lag) can be either very small or very large. Typically, the increase in latency indicates that the servers do not cope with the current load and they need to be scaled further, for example by techniques of horizontal and vertical shading .

Synchronous mode

Synchronous replication mode will ensure that data is copied to the Slave. 

This will simplify the work in the application, because All read operations can always be sent to the Slave. However, this can significantly reduce the speed of MySQL. Synchronous mode should not be used in Web applications.

"Manual" replication

It should be remembered that replication is not a technology, but a technique. Built-in replication mechanisms can bring unnecessary complications or do not have some necessary function. Some technologies do not have built-in replication at all.

In such cases, you should use an independent implementation of replication. In the simplest case, the application will duplicate all requests to several database servers: 

When writing data, all requests will be sent to several servers. But read operations can be sent to any server. The load will be distributed over all available servers:

<?
$dbs = [
	'10.10.0.1',
	'10.10.0.2'
];

foreach ( $dbs as $db )
{
	$connection = mysql_connect($db, 'root', 'pwd');
	mysql_query('INSERT INTO users ...', $connection);
}


# ...

$connection_read = mysql_connect($dbs[array_rand($dbs)], 'root', 'pwd');
mysql_query('SELECT * FROM users WHERE ...', $connection_read);
# All data modification operations occur on multiple servers, and readings occur on one random

This will allow you to take advantage of replication even if the technology itself does not support it.

Failure

If one of the servers fails in such a scheme, the following should be done:

Exclude the server from the list of used.
Configure Master-Slave replication on the new server using one of the working servers as the Wizard.
When all replication data is synchronized, enable the server back to the list used and stop replication.
The most important

Replication is used more to reserve databases and less for scaling. Master-Slave replication is convenient for distributing read requests to multiple servers. The manual replication approach will take advantage of replication for technologies that do not support it. Often, replication is used in conjunction with shading when solving scaling issues.