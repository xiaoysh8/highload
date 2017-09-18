Tuning base Postgres

The databases, as you may have noticed, are not limited to MySQL. There are others! Someone chooses Postgres for their products, and I generally support this choice and consider it reasonable. For heavily loaded systems, Postgres has a number of advantages over competitors, but this is another time.

And so, you postgres, you have an application (maybe in plans) with specific requirements for the number / size of tables, the record / read ratio, etc. Standard settings, as always suitable only for standard projects, but no standard projects. Therefore, the server will have to be tuned to your requirements to ensure its optimal performance.

First, a few words about the settings of Postgres. The configuration file is usually located at:

/etc/postgresql/8.3/main/postgresql.conf
Naturally, this is an example for version 8.3. Any configuration parameters belong to different levels, the list of which is given below:

Postmaster - requires server restart
Sighup - requires only the HUP signal (the running server will reload the configuration file without stopping work)
User - the value can be set within the session and is relevant only within this session
Internal - sets only at compile time, i.e. not changeable! (For reference only)
Backend - value must be set before the session starts
Superuser - can be changed while the server is running, but only from under the user with superuser rights
The first point is highlighted, because You will almost always encounter such parameters.

What you should pay attention to in the settings, and what values ​​should be changed:

listen_addresses

By default, Postgres only accepts connections from local services, because listens to the localhost interface . If you are planning a network subsystem consisting of more than one computer (your database server will be on a separate computer), you will need to change this parameter:

listen_addresses = '*'
Now postgres can accept connections from remote services via TCP.

max_connections

This parameter specifies the maximum number of simultaneous connections that the server will serve. In principle, this number should be determined based on the requirements for the system. This parameter has a greater effect on the use of resources. If you just start, set this value small (16 ... 32), gradually increasing it (as necessary - this measure will receive errors from postgres "too many clients").

Keep in mind! To support each active client, postgres spends a lot of resources, and if you need to achieve performance in several thousand active connections, then you should use connection managers, for example: Pgpool .

shared_buffers

This parameter specifies how much memory postgres will allocate for caching data. In standard delivery, the value of this parameter is scanty - to ensure compatibility. In practical conditions, this value should be set at 15..25% of all available RAM.

effective_cache_size

This parameter helps the postgres scheduler to determine the amount of available memory for disk caching. Based on whether the memory is available or not, the scheduler will make a choice between using indexes and using a table scan. This value should be set at 50% ... 75% of all available RAM, depending on how much memory is available for the system cache. Once again - this parameter does not affect the allocated resources - this is estimated information for the scheduler.

checkpoint_segments

You should pay attention to this setting if you have a lot of records in the database (for high-loaded systems this is a normal situation). Postgres writes data to the database in batches (WALL segments) - each with a size of 16Mb. After a certain number of such portions are recorded (determined by the checkpoint_segments parameter ), a checkpoint occurs. A checkpoint is a set of operations that postgres performs to ensure that all changes have been written to data files (hence, if a failure occurs, the recovery occurs at the last checkpoint). Performing checkpoints every 16Mb can be very resource intensive, so this value should be increased to at least 10 .

For cases with a large number of entries, it is worthwhile to increase this value from 32 to 256.

work_mem

An important parameter for queries that use all sorts of complex sampling and sorting. Increasing it allows you to perform these operations in RAM, which is much more efficient than on the disk (still). Be careful! This parameter specifies how much memory to allocate for each such operation! Therefore, if you have 10 active clients and each performs 1 complex query, then the value in 10MB for this parameter is 100MB of RAM. This parameter should be increased if you have a large amount of memory available. To start, you should set it to 1Mb.

maintainance_work_mem

This parameter determines the amount of memory for various statistical and control processes (for example, evacuation). The developers recommend to allocate 128 ... 256MB for these needs.

wal_buffers

This parameter should be increased in systems with a large number of records. The value in 1MB is recommended by postgres developers even for very large systems.

synchronous_commit

Pay special attention to this parameter! It enables / disables synchronous writing to the log files after each transaction. This protects against possible loss of data. But this imposes a restriction on the bandwidth of the server.

Suppose, in your system, the potentially low possibility of losing a small number of changes when a system crashes is not critical. But it is vitally important to provide several times more performance by the number of transactions per second. In this case, set this parameter to off (disabling synchronous recording).

The most important

The performance of Postgres can be significantly increased, we will configure several parameters. Make sure that the database is configured for your hardware configuration.