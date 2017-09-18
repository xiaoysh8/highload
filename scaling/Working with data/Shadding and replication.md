Shadding and replication

Scaling of databases is the most difficult task during the growth of the project. 90% of all efforts are usually just for work related to the growth in the volume of data and operations with them. The classical scheme of the application with the database looks like this: 

One database server at some point stops coping with the load. At this point, you should apply the scaling techniques described here.

Before you start to scale, you need to analyze slow queries and make sure that the MySQL server is configured optimally .

Strategies

At the heart of data scaling lies the same principle as the basis for scaling Web applications. This is the separation of data into groups and their allocation to separate servers. There are two main strategies - replication and sharding.

Replication

Replication allows you to create a complete database duplicate. So, instead of one server you will have several of them: 

Master-slave

Most often use the master-slave scheme:

Master - this is the main database server, where all data is received. All changes to the data (addition, update, deletion) must occur on this server.
Slave is an auxiliary database server that copies all data from the master. From this server should read the data. There can be several such servers.
Replication allows you to use two or more identical servers instead of one. Data read operations (SELECT) are often much larger than data modification operations (INSERT / UPDATE). Therefore, replication allows you to unload the primary server by transferring the read operations to the slave.

Working from the application

In the application, you will have two connections to the database. One for the master and one for the slave:

<?
$master = mysql_connect('10.10.0.1', 'root', 'pwd');
$slave = mysql_connect('10.10.0.2', 'root', 'pwd');


# какой-то код и все такое...

$q = mysql_query('INSERT INTO users ...', $master);


# еще какой-то код...

$q = mysql_query('SELECT * FROM users WHERE...', $slave);
# When you run queries, you must use the appropriate connection

Replication is usually supported by the DBMS itself (for example, MySQL) and is configurable regardless of the application. Read more about the configuration, use, and types of data replication in the MySQL example.

It should be noted that replication itself is not a very convenient scaling mechanism. The reason for this - the dissynchronization of data and the delay in copying from the master to the slave. But this is an excellent tool for providing fault tolerance. You can always switch to a slave if the master breaks and vice versa. Most often, replication is used in conjunction with shading precisely for reasons of reliability.

Sharding

Sharding (sometimes shadding) is another technique for scaling work with data. Its essence is in partitioning (partitioning) the database into separate parts so that each of them can be carried to a separate server. This process depends on the structure of your database and is executed directly in the application as opposed to replication: 

Vertical Sharding

Vertical sharding is the allocation of a table or a group of tables to a separate server. For example, in an application there are such tables:

users - user data
photos - user photos
albums - albums of users
You leave the users table on one server, and transfer the photos and albums to another table . In this case, in the application, you will need to use the appropriate connection to work with each table:

<?
$users_connection = mysql_connect('10.10.0.1', 'root', 'pwd');
$photos_connection = mysql_connect('10.10.0.2', 'root', 'pwd';


# какой-то код и все такое...

$q = mysql_query('SELECT * FROM users WHERE ...', $users_connection);


# еще какой-то код...

$q = mysql_query('SELECT * FROM photos WHERE...', $photos_connection);


# еще какой-то код...

$q = mysql_query('SELECT * FROM albums WHERE...', $photos_connection);
# There will be a separate connection for each table or group of tables

Unlike replication, we use different connections for any operations, but with certain tables. Read more about using vertical shading in practice.

Horizontal Sharding

Horizontal sharding is the separation of one table into different servers. This should be used for huge tables that do not fit on one server. The division of the table into pieces is done according to this principle:

Multiple servers create the same table (only the structure, without data).
The application selects the condition by which the desired connection will be determined (for example, even for one server, and odd for another).
Before each access to the table, the desired connection is selected.
Let's say our application is working with a huge table that stores user photos. We have prepared two servers (usually called shards ) for this table. For odd users, we will work with the first server, and for even users - with the second server. Thus, on each of the servers there will be only a part of all data about the users' photos. It will look like this:

<?

# список соединений для таблицы с фотками
$photo_connections = [
	'1' => '10.10.0.1',
	'2' => '10.10.0.2',
];

$user_id = $_SESSION['user_id'];


# получение фотографий для пользователя $user_id
$connection_num = $user_id % 2 == 0 ? 1 : 2;
$connection = mysql_connect($photo_connections[$connection_num], 'root', 'pwd');
$q = mysql_query('SELECT * FROM photos WHREE user_id = ' . intval($user_id), $connection);
# Before accessing the table, we select the connection we need

The result of this operation $ user_id% 2 is the remainder of the division by 2. That is, for even numbers - 0, and for odd ones - 1.

Any work with the photos table will now occur only after obtaining the necessary connection based on $ user_id .

Horizontal shadding is a very powerful tool for scaling data. But at the same time it's very non-trivial. Read more about using horizontal shading in practice.

Do not apply the technique of shading to all tables. The right approach is a step-by-step process of splitting up growing tables. You should think about horizontal shading, when the number of records in one table goes from tens of millions to hundreds of millions.

Sharing

Shading and replication are often used together. In our example, we could use two servers per shard table:

photos_master_1 - master of the first half of the table.
photos_slave_1 - slave of the first half of the table.
photos_master_2 - master of the second half of the table.
photos_slave_2 - slave of the second half of the table.
Then in the application work with this tablet can look like this:

<?

# список соединений, для каждого шарда - мастер и слейв
$photo_connections = [
	'1' => [
		'master' => '10.10.0.10',
		'slave' => '10.10.0.11',
	],
	'2' => [
		'master' => '10.10.0.20',
		'slave' => '10.10.0.21',
	],
];

$user_id = $_SESSION['user_id'];


# Читаем данные со слейвов
$connection_num = $user_id % 2 == 0 ? 1 : 2;
$connection = mysql_connect($photo_connections[$connection_num]['slave'], 'root', 'pwd');
$q = mysql_query('SELECT * FROM photos WHREE user_id = ' . intval($user_id), $connection);


# какой-то код...


# Изменение данных происходит на мастерах
$photo_id = 7;
$connection_num = $user_id % 2 == 0 ? 1 : 2;
$connection = mysql_connect($photo_connections[$connection_num]['master'], 'root', 'pwd');
$q = mysql_query('UPDATE photos SET views = views + 1 WHREE photo_id = ' . intval($photo_id), $connection);

# Read the data from the slaves, and write to the master server

This scheme is often used not to scale, but to provide fault tolerance. So, if one of the shard servers fails, there will always be a spare one.

Key-value of the database

It should be noted that most Key-value databases support platform-level shading. For example, Memcache. In this case, you just specify a set of servers for the connection, and the platform does the rest:

<?
$m = new Memcache;
$m->addServer('10.5.0.1');
$m->addServer('10.5.0.2');
$m->addServer('10.5.0.3');
...
$m->get('user1')
#Memkesh himself is able to determine the right server for each key

The most important

Shading and replication are popular and powerful techniques for scaling data management systems. Despite the examples for MySQL, these approaches are universal and can be applied for any technology.

Remember , the process of data scaling is an architectural solution, it is not related to a particular technology. Do not make mistakes of our fathers - do not move from a technology known to you to a new one because of support or not support of shading. Problems are usually related to architecture, not to a specific database.