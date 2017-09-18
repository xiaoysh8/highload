Data re-balancing with shading

With shading , you inevitably need to rebalance the data. Precisely predict the growth of the volume and form of data is almost impossible. Therefore, data rebalancing is the same systematic operation as data storage. It must be planned at the design stage, and not at the administration stage.

Let the data be distributed among 5 servers. For example, this is a table of private messages:

messages : id | user_id | time | message | sender
The data in this table is distributed by checking user_id (the remainder of the division by 5):

Remainder of the division	user_id% 5 == 1	user_id% 5 == 2	user_id% 5 == 3	user_id% 5 == 4	user_id% 5 == 0
Server number with data	1	2	3	4	5
Those. to get a list of the user's private messages with id = 47, you need to perform the following operations:

<?

$user_id = 47;
$server = $user_id % 5 ? : 5;


# подключаемся к нужному серверу
$connection = connect($server);


# выполняем обычный запрос (в рамках подключения)
$messages = query($connection, "SELECT * FROM messages WHERE user_id = {$user_id}"); # без инъекций
# Choosing a database server for a particular user under shading

Preparation for rebalancing

If the current servers are busy at 70 ... 80% this is a sign that it's time to expand their number. How many new servers? It is possible to estimate very simply:

The amount of occupied disk space (on all servers) should not be more than 50%, otherwise the readiness for explosive loads is very low.
The speed of data growth should not lead to a re-balancing faster than you have time to prepare a new iron. From several days to a week usually. But it is clear that it is better to provide the stock (for example, a month).

Suppose we put another 5 servers and now we have 10. If we are talking about writing new data, then there is nothing to do - just change the logic of the shading (the remainder of the division should now be counted from 10). But for correct retrieval of previously stored data, it is necessary to rebalance the data from old servers to new ones.

For example, in the old scheme with 5 servers, user messages with id = 47 were on the server:

[ 5 servers]: user_id = 47, server_id = 2
# remainder from division 47 to 5 = 2

In the new scheme with 10 servers, the data should lie here:

[ 10 servers]: user_id = 47, server_id = 7
# remainder from division 47 to 10 = 7

Rebalancing

There are two strategies. Synchronous and asynchronous.

Asynchronous rebalancing

After installing new servers, before switching them on, you need to move all data from old nodes to new ones. This is a mega script that does the following:

<?

foreach ( $users as $id )
{
	# вычисляем номера серверов в старой и новой схеме
	$old_server = $id % 5 ? : 5;
	$new_server = $id % 10 ? : 10;

	if ( $old_server != $new_server )
	{
		# получаем все сообщения со старого сервера
		$all_messages = get_messages($old_server, $id);

		# сохраняем все сообщения на новый сервер
		save_messages($new_server, $id, $all_messages);
	}
}
After moving all data, the application will start working with a new set of servers. 

It is clear that for the time of moving it is necessary to somehow know which of the users has already been processed and who else is not. This can be done by adding a flag to the user's table and resetting it before rebalancing:

users : id | email | ... | messages_rebalanced
Then, during the re-balancing, you will need to update this checkbox:

<?

foreach ( $users as $id )
{
	# перемещаем данные со старого на новый сервер
	# ...
	
	# сохраняем статус пользователя
	save_user($id, ['messages_rebalanced' => 1]);
}
In the application, this will ensure normal operation right during data movement:

<?

$user_id = 47;

if ( $user['messages_rebalanced'] )
{
	# данные уже перемещены, читаем с нового сервера
	$server = $user_id % 10 ? : 10;
}
else
{
	# данные еще не перемещены, читаем со старого сервера
	$server = $user_id % 5 ? : 5;
}
# on the checkbox we find out from which server it is worth reading the data for the user

After rebalancing, we remove the check-flag code (now it will be installed for all users). We reset the flag itself to zero in the database, so as not to forget it the next time.

Synchronous rebalancing

Synchronous rebalancing implies that we do not do anything with the data until a request comes to them. For example, the user data c id = 47 will be on the 2nd server, although there will already be 10 servers.

So, we add new servers and immediately include them in the work.

If a write request comes up, we write the data to the correct server in the new schema.

However, as soon as the read request comes, we check the old schema and move the data from the old server to the new one: 

Suppose we want to read data from the user user_id = 47, the data of which are still on the old server (this we also learn the checkbox messages_rebalanced ). Then before we get the data, we will do the following:

<?

$user_id = 47;

if ( !$user['messages_rebalanced'] )
{
	# перемещаем сообщения перед их чтением
	$old_server = $user_id % 5 ? : 5;
	$messages = get_messages($server, $user_id);

	$new_server = $user_id % 10 ? : 10;
	save_messages($new_server, $user_id, $messages);

	save_user($user_id, ['messages_rebalanced' => 1]);
}


# читаем все сообщения с нового сервера
$server = $user_id % 10 ? : 10;
$messages = get_messages($server, $user_id);
# before the first reading in the new schema, the data will be moved from the old to the new server

This method is obviously more convenient and efficient than the asynchronous method. Only the actual data will be moved, except that the whole process will be distributed in time.

but, be careful! When moving large amounts of data (for example, if messages are stored for years and can deal with hundreds of megabytes), it is better to use asynchronous rebalancing. Otherwise, the application may be extremely slow for the user during the move operations.

TL; DR

Rebalancing is part of the design. It will have to be done. Prepare to have a convenient rebalancing mechanism for the first time. Simpler to use the synchronous method. However, for large amounts of data being moved, it is worth choosing asynchronous data.