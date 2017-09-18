Horizontal Sharding

One way or another, there is a situation where the database server eventually has to work with huge tables. 

In such cases, you can use a special technique of horizontal shading . In this case, a large table is divided into several parts. Each part of the table is placed on a separate server.

Preparation

For the introduction of horizontal shading it is necessary to prepare several identical database servers. On each of them, the structure of the selected table should be created. Work on the distribution of data rests on the application itself.

Distribution of data

First of all, the data in large tables must be somehow shared between the servers. To do this, you need to define the separation condition. Imagine that we share the photos table , which stores the pictures of users. It has the following structure:

id
user_id
date
photo
We could use the user_id column to separate the data in this table. Then we could save all photos for even users to one server. And for odd ones, on the other:

<?
$cons = ['10.10.0.1', '10.10.0.2'];

$user_id = $_SESSION['user_id'];
$con_num = $user_id % 2 == 0 ? 0 : 1;
$con = mysql_connect($cons[$con_num]);
mysql_query('INSERT INTO photos ...', $con);
# choose different connections - for even and odd values $ user_id

When fetching data for a specific user, we also need to define the appropriate connection:

<?
$cons = ['10.10.0.1', '10.10.0.2'];

$user_id = $_SESSION['user_id'];
$con_num = $user_id % 2 == 0 ? 0 : 1;
$con = mysql_connect($cons[$con_num]);
mysql_query('SELECT FROM photos ...', $con);
# For any calls to the photos table, you must select the desired connection

Separation into n servers

The remainder of the division is well suited for the uniform distribution of records by any number of servers. So, if we have 7 shards, we will use the remainder of the division by 7:

<?
$cons = ['10.10.0.1', '10.10.0.2', '10.10.0.3', '10.10.0.4', '10.10.0.5', '10.10.0.6', '10.10.0.7'];

$user_id = $_SESSION['user_id'];
$con_num = $cons[ $user_id % 7 ];

# ...
# The remainder of the division allows you to conveniently share data between servers

If there are 50 servers, we will use the remainder of 50, etc. Thus, we can evenly divide any tables by the condition of the remainder from dividing the value of some column. For this type of division, tables that have a connection to the parent object are well suited:

pictures of users, we divide by user_id
user messages, divide by user_id
article comments, share on article_id
products in the category, divide by category_id
...
Dictionary

Sometimes it is not possible to divide the data into a digital column. Let's say we have a very large table with news that we want to divide into several servers (shards):

id
title
body
In this case, you should use a dictionary. This is another table in which the relationship between the news ID and the shard number will be indicated . At the time of adding news, we will select a random shard and write its number in the dictionary:

<?
$dict_con = mysql_con('10.10.0.1');

$cons = ['10.10.0.2', '10.10.0.3'];
$con_num = array_rand($cons);
$con = mysql_connect($cons[$con_num]);


# сохраняем новость на случайный шард
mysql_query('INSERT INTO news SET title = ...', $con);
$id = mysql_insert_id($con);


# сохраняем номер шарда для этой новости
mysql_query('INSERT INTO news_shards SET news_id = ' . $id . ', shard_id = ' . $con_num, $dict_con);
# The $ dict_con connection is used for the dictionary

Then, to read some news, you will first need to get the shard number from the dictionary:

<?
$dict_con = mysql_connect('10.10.0.1');
$q = mysql_query('SELECT shard_id FROM news_shards SET news_id = ' . $id . ', $dict_con);
$con_num = mysql_fetch_assoc($q)['shard_id'];

$cons = ['10.10.0.2', '10.10.0.3'];
$con = mysql_connect($cons[$con_num]);

mysql_query('SELECT * FROM news WHERE id = ' . $id, $con);
# Get the shard number, and then the news

Restrictions

When using horizontal shading, there are a number of limitations.

Recent Entries

If the table is divided into different servers, it is impossible to make a general sample from it. For example, you can not get a list of the last ten photos or news from the examples above. If necessary, an additional table should be used, which will contain only the last 10 photos (or news). To insert there the data will be necessary at each addition of a photo:

<?
$cons = ['10.10.0.2', '10.10.0.3'];

$user_id = $_SESSION['user_id'];
$con_num = $user_id % 2 == 0 ? 0 : 1;
$con = mysql_connect($cons[$con_num]);
mysql_query('INSERT INTO photos SET ...', $con);
$id = mysql_insert_id($con);


# ID новой фотки запишем в таблицу свежих фоток
$new_con = mysql_connect('10.10.0.1');
mysql_query('INSERT INTO new_photos SET photo_id = ' . $id, $new_con);
# Save ID of new photos to a separate table on one (main) server

To ensure that this table does not become very large, it should be constantly cleaned. For example, leave only the last 100 entries:

<?

# ...
$id = mysql_insert_id($con);
mysql_query('INSERT INTO new_photos SET photo_id = ' . $id, $new_con);
mysql_query('DELETE FROM new_photos ORDER BY photo_id DESC LIMIT 100, 110', $new_con);
# The second request will permanently delete all records beyond the 100th

Search and Filtering

Search and filtering according to the table data should be carried out using a suitable technology, such as Elastic Search . Indexing data in this case will occur immediately on all shards of the table.

In addition, you can use the preparation of data to search in advance for every possible sample. For example, we can search for photos of different sizes. Then, you need to determine in advance the groups of possible sizes of photos:

big
medium
small
For each size, you can create a separate table that will store the ID and other necessary photo data. When you insert a photo into these tables, the corresponding IDs will appear:

<?

# ...
$size = 'big';
$con = mysql_connect($cons[$con_num]);
mysql_query('INSERT INTO photos SET ...', $con);
$id = mysql_insert_id($con);


# сохраним данные о новой фотке в соответствующую таблицу поиска
$con_search = mysql_connect('10.10.0.10');
mysql_query('INSERT INTO photos_' . $size . ' SET photo_id = ' . $id, $con_search);
# Save ID of new photos to a separate table on one (main) server

Next, we can very easily make a selection of all the photos of the desired size:

<?

# ...
$sizes = ['big', 'medium', 'small'];
$size = $sizes[ $_GET['size'] ];

$con_search = mysql_connect('10.10.0.10');
$q = mysql_query('SELECT * FROM photos_' . $size . ' LIMIT 10 ', $con_search);

# ...
# select the required table to search by the given criteria

Rebalancing

When adding new shards, it is necessary to rebalance the data. So, if the shards were 2, and it was 3, some records should be moved to a new shard. 

In order to rebalance the data in a running application, it is necessary to use two sets of shards - old and new. For each record, it is necessary to store the status of its distribution - whether there has been a rebalancing or not. At the time of data sampling, we will check this status. If the data is not yet moved, we will move them and put a mark in the status. The status is conveniently stored in a key-value database or in a separate table. For an example with pictures of users:

<?

# старый набор шардов
$cons_old = ['10.10.0.2', '10.10.0.3'];


# новый набор шардов
$cons_new = ['10.10.0.2', '10.10.0.3', '10.10.0.4'];

$user_id = $_SESSION['user_id'];


# получаем статус перебалансировки фоток для пользователя из redis
$rebalanced = redis::get('user_photos_sharding' . $user_id);


# получаем соединение для нового набора шардов
$con_num = $user_id % 3 == 0 ? 0 : 1;
$con = mysql_connect($cons_new[$con_num]);


# данные необходимо перебалансировать?
if ( !$rebalanced )
{
	# получаем соединение для старого набора шардов
	$con_num_old = $user_id % 2 == 0 ? 0 : 1;
	$con_old = mysql_connect($cons_old[$con_num_old]);

	# перебалансируем данные только если старый и новый шард отличаются
	if ( $con_num_old != $con_num )
	{
		$q = mysql_query('SELECT * FROM photos WHERE user_id = ' . $user_id, $con_old);
		
		# копируем все фотки со старого шарда на новый
		while ( $row = mysql_fetch_assoc($q) ) mysql_query('INSERT INTO photos SET ...', $con);
	
		# удаляем все фотки со старого шарда
		mysql_query('DELETE FROM photos WHERE user_id = ' . $user_id, $con_old);
	}

	# ставим отметку о том, что перебалансировка проведена
	redis::set('user_photos_sharding' . $user_id, true);
}


# делаем выборку согласно новому набору шардов
mysql_query('SELECT * FROM photos WHERE user_id = ' . $user_id, $con);
# rebalance right at the time of sampling

fault tolerance

When the number of shards increases, the probability of failure of one of them increases. To ensure fault tolerance, each shard must be backed up using Master-Slave replication . 

Partitioning

MySQL also supports partitioning . This is the ability to split the table into different logical groups within the same server. Partitioning can improve the efficiency of working with large tables, when most operations are performed only with fresh data (ie in the "top" of the table). This approach works only within a single server .

The most important

Horizontal shading is one of the most powerful tools for scaling a database. Splitting the table into separate servers allows you to scale them almost infinitely. Introduce shading gradually and start only with the largest tables. Use vertical shading to distribute the load between groups of tables.