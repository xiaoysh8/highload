Vertical Sharding

Typically, Web applications work with a single database server. Almost always, the application uses more than one table. 

One of the techniques for scaling a database is to partition tables across different servers. In this case, several tables will be on the same server, and the rest on the other. Then requests to different tables will be processed by different database servers. This is called vertical shading . 

Servers with different tables are called shards . Some tables are usually larger than others. Sharding usually starts with the largest and heaviest tables. They are allocated to a separate group and carried to a separate server.

Preparation of shading

To select a table on a separate server, you need to perform a few simple steps.

1. A separate connection

Make sure that the code for all calls to the selected table uses a separate connection. Before enabling a separate server, it will simply duplicate the primary connection:

<?
$con = mysql_connect('10.10.0.1');
$con_photos = mysql_connect('10.10.0.1');

#...
mysql_query('SELECT * FROM users ...', $con);

#...
mysql_query('SELECT * FROM photos ...', $con_photos);
# Duplicate connections for selected tables

2. Making a copy

Next, you need to create a complete copy of the selected tables on the new server. In a simple case, you can use a dump and stop the entire site for the period of the copy creation. To do this without a pause, you should use replication . In this case, the replica of the required tables is configured on the new server. As the Master, the old server will act, and the new one will be Slave.

3. Toggle connection

After that, just switch the connection to a new server:

<?
$con = mysql_connect('10.10.0.1');
$con_photos = mysql_connect('10.10.0.2');

#...
# We now use two different connections for different tables

If you used replication to create a copy, you must stop it.

Working from the application

In the application, we will work with different connections for different tables:

<?
$con = mysql_connect('10.10.0.1');
$con_photos = mysql_connect('10.10.0.2');

#...
mysql_query('SELECT * FROM users ...', $con);

#...
mysql_query('SELECT * FROM photos ...', $con_photos);
# we use different connections for the corresponding tables

This means that we will have as many connections as there are shards. Their number may be large, so it's better to use the lazy resource loading technique to establish connections.

JOINs

It is clear that JOIN two tables on different servers can not be done. There are two ways to solve this problem.

Groups of tables

Often JOIN requests take place only between a group of tables that are logically related to each other. For example, tables that store data about albums and photos of users:

photos list of photos, contains album_id
albums list of albums
In this case, it is more convenient to put a whole group of these tables on a separate shard at once. This will allow JOIN to be used within this group.

Selection in the application

In another version, the functionality of JOIN'a will have to be transferred to the application. For example, such a query:

SELECT * FROM photos p JOIN albums a ON (a.id = p.album_id) WHERE a.user_id = 1
# Select all photos from user 1

If the users and albums tables are on different servers, you can get the same result like this:

<?

# ...
$q = mysql_query('SELECT * FROM albums WHERE user_id = 1', $connection_albums);
$albums = mysql_fetch_all($q);


# получаем список ID альбомов пользователя
foreach ( $albums as $album ) $album_ids[] = $album['id'];


# выбираем все фотки для указанных альбомов
$q = mysql_query('SELECT * FROM photos WHERE album_id IN (' . implode(',', $album_ids) . ')', $connection_photos)
# Do two requests instead of one JOIN'a

fault tolerance

The probability of failure of database servers increases with the increase in their number. 

To ensure fault tolerance, you must back up the database servers using replication . In this case, each shard will have a backup server with a copy of the data.

In the event of failure of one of the shards, it will be sufficient to switch its connection to the standby server:

<?
$con = mysql_connect('10.10.0.1');
$con_photos = mysql_connect('10.10.0.2');
$con_photos = mysql_connect('10.10.0.3');

#...
# To use the backup server it will be enough to change the connection parameters

The most important

Vertical sharding is a convenient mechanism for scaling databases. Selecting logically linked groups of tables into separate shards will even allow you to use JOINs. Be sure to use the redundancy scheme to increase the resiliency in shading. Start with the largest and heaviest tables. For particularly large tables, use the horizontal shading technique .