Optimal Mysql Configuration

The default configuration parameters in Mysql are designed for microscopic databases that work under low loads on modest hardware.

Setting some parameters can improve database performance hundreds of times!
The process of optimal configuration Mysql consists of two parts - the initial adjustment and adjustment of parameters during operation. Adjusting the parameters in the operating mode largely depends on the specifics of your system and its monitoring. Let's look at the parameters and recommendations for setting their values.

You need to make the settings in my.cnf .

innodb_buffer_pool_size

If you use only InnoDB tables, set this value to the maximum possible for your system. The InnoDB buffer caches both data and indexes. Therefore, the value of this key should be set at 70% ... 80% of all available memory.

innodb_buffer_pool_size = 24G
# Given that on our server 32GB of RAM

innodb_log_file_size

This option affects the recording speed. It sets the size of the operation log (since operations are first written to the log, and then applied to the data on the disk). The more this log, the faster the records will work (because they fit more into the log file). There are always two files, and their size is the same. The value of the parameter is the size of one file:

innodb_log_file_size = 512M
# So two files will give the size of the log to 2x512M = 1G

It is worthwhile to understand, that an increase in this parameter will also increase the recovery time of the system in case of failures. This is because when the system starts, all the data from the logs will be rolled into the data. However, with each new version, the performance of this process is growing. Think about using replicas to ensure availability, so as not to depend on the time to restore the database.

innodb_log_buffer_size

This is the size of the transaction buffer, which was not yet clogged. The value of this parameter should be changed in cases, if you use large fields like BLOB or TEXT.

 innodb_log_buffer_size = 2M
# Default values ​​in 1M should be sufficient for most cases

innodb_file_per_table

If you enable this option, Innodb will save the data of all tables in separate files (instead of one file by default). The increase in productivity will not be, however there are a number of advantages:

When you delete the tables, the disk will be freed. By default, the shared data file can only expand, but not decrease.
Using the compression format of the tables will require this option.
innodb_file_per_table = ON
# Since version 5.6, this option is enabled by default

innodb_flush_method

This parameter specifies the logic for data reset to disk. In modern systems, when using RAID and reserve links, you will choose between O_DSYNC and O_DIRECT :

innodb_flush_method = O_DSYNC
# Be aware of the mandatory use of redundant nodes (for example, replicas )

innodb_flush_log_at_trx_commit

Changing this parameter can increase the bandwidth of writing data to the database in hundreds of times. It determines whether Mysql will reset each operation to disk (to the log file).

Here you should follow this logic:

innodb_flush_log_at_trx_commit = 1 for cases where data integrity is priority number one.
innodb_flush_log_at_trx_commit = 2 for cases where a small loss of data is not critical (for example, you use duplication and will be able to recover a small loss). In this case, the transactions will be reset to the log on the disk only once per second.
Set the value to your own discretion, however in most cases the second option is suitable:

innodb_flush_log_at_trx_commit = 2
# Significant acceleration of writing to the database, but this will require data duplication mechanisms

query_cache_size

The value of this parameter determines how much memory should be used for the query cache. The most correct approach is not to rely on this mechanism. In practice, it works very inefficiently. For example, the entire query cache for a specific table is reset whenever there is at least one change to the table. This can cause caching to even slow downdatabase:

query_cache_size = 0
# However, make sure you use indexes to ensure high-speed query performance

max_connections

Do not change the value of this parameter at the start. However, if you get errors "Too many connections" , this option should be raised. It defines the maximum number of simultaneous connections to the database:

max_connections = 256
# Raise the value gradually when connection errors occur

TL; DR

The default settings probably will not work. Therefore, it is necessary to go through the specified parameters in the article and select values ​​for them. If absolutely laziness - generator settings Mysql .