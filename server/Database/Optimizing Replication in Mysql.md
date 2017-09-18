Optimizing Replication in Mysql

In databases, replication is often used to ensure high availability. However, the replication process can load the backbone of the server (master). In addition, even the same in terms of resources, the cue can lag behind the master. Let's see.

Monitoring

After configuring the replication, you should constantly monitor several parameters on the slave:

mysql -e "SHOW SLAVE STATUS \ G"
# The query returns statistics and slave settings

It is worth paying attention to such indicators:

               Slave_IO_State: Waiting for master to send event
			     ...
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
			     ...
                   Last_Errno: 0
                   Last_Error: 
			     ...
        Seconds_Behind_Master: 0 
			     ...
# The status of the replica

The Slave_IO_Running and Slave_SQL_Running indicators reflect the normal operation of the replica. Both must have meaningYes. When one of these indicators is equal toNo, the text of the replication error will be seen in Last_Error .

The Seconds_Behind_Master parameter reflects the number of seconds that the slave lags behind the wizard. This indicator should be equal tozero (sometimes it can grow up to several seconds).

Disable sync_binlog

The sync_binlog parameter specifies the logic for synchronizing data from the disk with the disk. If the value is 1 , write to disk will occur after each transaction. This makes the repository very reliable, but extremely heavily loads the disk subsystem on the wizard.

A value of 0 will disable synchronization from Mysql, and the database will rely on the OS in question to write the log to disk. This value can increase the performance of the wizard several times.

You can check the current value as follows:

mysql -e "show variables like 'sync_binlog'"
# Checking the synchronization mode of the binologist

+ --------------- + ------- +
| | Variable_name | Value |
+ --------------- + ------- +
| | sync_binlog | 1      |
+ --------------- + ------- +
# It's better to disable sync

You can disable synchronization without rebooting the server, just do this:

SET GLOBAL sync_binlog = 0;
However, do not forget to fix this parameter in my.cnf too , so that it will be preserved after the reboot:

...
[mysqld]
sync_binlog = 0
...
In .io we always disable synchronization of the binologist. This increased the throughput of our Mysql nodes in 2 ... 3 times, unloading the disk subsystems of the masters.

Multithreaded replication

Until recently, the Mysql replica worked in only one thread. Then, even if the master and slave are identical in characteristics, the slave can still lag behind the master.

In the version Mysql 5.6introduced support for parallel replication . It allows the slave to process the binlog in parallel in several threads. This mode is enabled by setting it in my.cnf on the slave:

...
[mysqld]
slave-parallel-workers = 2 
...
# Enable replication in 2 threads

The number specifies the number of threads and can take values ​​from 2 to 1024 . A value of 0 will disable the multithreading of the binlog.

When you enable this setting, Mysql will distribute the processing of the binolog of different databases between different threads. This will enable Mysql not to wait for operations to be completed in different databases, but to execute them in parallel. On the wizard, you do not need to configure anything.

It is clear that if you have only one database, you will not get a performance boost. But in the versionMysql 5.7for you can change the type of distribution of operations using the setting in my.cnf :

...
[mysqld]
slave-parallel-workers = 2
slave-parallel-type = LOGICAL_CLOCK
...
# Changes to the parallelization type of the binlog processing

In this case, all transactions (more precisely, zakomichnye transactions) from the binlog will be processed in parallel.

TL; DR

Monitor Seconds_Behind_Master on the slave, it must be zero.
Disable synchronization of the binology on the wizard: sync_binlog = 0 .
Use the parallel processing of the binolog on the slave: slave-parallel-workers = 2 (or other number of threads) in new versions of Mysql.