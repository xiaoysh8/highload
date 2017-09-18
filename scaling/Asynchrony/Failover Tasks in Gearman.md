Failover Tasks in Gearman

The Gearman queue system uses only RAM to store tasks by default. This means that when the server is rebooted or suddenly shut down, all tasks from the queue will be lost. 

Permanent storage

Gearman allows you to use persistent storage in order to solve the problem of losing tasks in case of failures. It works like this:

When a new task is received, Gearman saves its data to a persistent store with a unique identifier.
After processing the task with the vorker, Gearman removes its data from the persistent storage using a unique identifier.
At boot, Gearman will check the persistent storage. If there are detected tasks, it will load them into the queue.
This means that for any failure, all tasks will already be saved to disk. And the next time they boot, they will be restored. In this simple way, you can implement protection from losing tasks. 

MySQL

Gearman supports several repositories, incl. MySQL. First you need to create a separate database and a special structure table:

CREATE TABLE `queue` (
  `unique_key` varchar(64) DEFAULT NULL,
  `function_name` varchar(255) DEFAULT NULL,
  `priority` int(11) DEFAULT NULL,
  `data` longblob,
  `when_to_run` bigint(20) DEFAULT NULL,
  UNIQUE KEY `unique_key` (`unique_key`,`function_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
# The table in which Gearman will save tasks

unique_key specifies the unique identifier for the task. Its length is determined by the constant GEARMAN_UNIQUE_SIZE (the default is 64). function_name is the name of the queue. Both of these parameters define a unique task in the queue table.

After that, you need to start the Gearman server with the following parameters:

gearmand --queue-type = MySQL --mysql-host = localhost --mysql-port = 3306 \
         --mysql-user = gearman --mysql-password = frCBRFup4QzAD4wP \
         --mysql-db = gearman --mysql-table = queue
# We assume that the queue table is stored in the gearman database

Everything else Gearman will do himself. The data in the queue table will have this structure:

mysql> select * from queue limit 5;
+ -------------------------------------- + ---------- ----- + ---------- + ------------- + ------------- +
| | unique_key | function_name | priority | data | when_to_run |
+ -------------------------------------- + ---------- ----- + ---------- + ------------- + ------------- +
| | 1d337490-3c01-11e4-a3d8-040118037e01 | sendmail | 1 | ["bla bla"] | 0 |
| | 366a1eac-3c04-11e4-a3d8-040118037e01 | sendmail | 1 | ["bla bla"] | 0 |
| | 366aa570-3c04-11e4-a3d8-040118037e01 | sendmail | 1 | ["bla bla"] | 0 |
| | 366b1da2-3c04-11e4-a3d8-040118037e01 | sendmail | 1 | ["bla bla"] | 0 |
| | 366b90de-3c04-11e4-a3d8-040118037e01 | sendmail | 1 | ["bla bla"] | 0 |
+ -------------------------------------- + ---------- ----- + ---------- + ------------- + ------------- +
5 rows in set (0.00 sec)
In the correct use case, the size of the queue table should tend to zero. Those. Vorkers should handle the incoming tasks as quickly as possible.

The most important

Use the persistent storage to ensure the integrity of tasks in Gearman. In addition to MySQL, PostgreSQL and SQLite are also supported. Do not forget to optimize the database , so as not to reduce the performance of the queue system.