How do I copy the selected Redis keys to another server?

Redis is a simple, fast and convenient key-value data store. When you scale any application, you need to transfer data between servers. Sometimes you do not need to transfer the entire database, but only a few selected keys.

Verifying the version of redis

The first thing you need to do is make sure that both servers have the same version of Redis, execute the command:

redis-server --version
In response we will receive information about the version:

Redis server v=3.0.3 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 build=58013d157c63182b
If the versions do not match, then you need to update the redis on one of the servers, then go directly to the transfer. In redis, the data can be transferred in several ways, we will consider two of the simplest. For the sake of convenience, the server from which we transfer is called A, the server where we transfer B.

Establishing communication between redis servers

For the first method, you need to establish a connection between the servers. We go to server B and open the config redis which is on the path /etc/redis/redis.conf , find the parameter bind and change the value to 0.0.0.0 .

To test the connection between servers, run the following command on server A:

redis-cli -h $target_host -p $target_port
$ target_host and $ target_port we take from the server B, in response to the start of the command should get the inscription PONG

Immediately transfer of data

Then on server A we start the transfer command:

redis-cli --scan --pattern '$pattern' | while read key; do echo "Copying $key"; 
redis-cli --raw -h $source_host -p $source_port -n $source_db DUMP "$key" | head -c -1|
redis-cli -x -h $target_host -p $target_port -n $target_db RESTORE "$key" 0; done
$ pattern - a regular expression by which we select keys;

$ source_ * - server A parameters;

$ target_ * - server B parameters;

After that check the presence of keys on both servers and change the bind parameter back to 127.0.0.1

Backup to files and restore via restore

For the second method it is necessary to do:

1. On server A, to make backup keys redis in files, this is simply done using the command:
redis-cli --scan --pattern "$pattern" | while read key; do echo "Copying $key"; 
redis-cli --raw -h $source_host -p $source_port -n $source_db DUMP "$key" | head -c -1 > move_$key; done
We 'll get many files with the prefix move_ it's our redis keys that match $ pattern .
2. Transfer them to server B.
3. On server B, you must run the restore command for redis:
cat $file_name | redis-cli -x restore $key_name 0;
To restore all files at once we can use the command:
for f in ~/move_t*; do cat $f | redis-cli -x restore "${f/\/root\/move_t/t}" 0; done
TL; DR

You can transfer the selected redis keys to another server using a simple command:

redis-cli --scan --pattern '$pattern' | while read key; do echo "Copying $key"; 
redis-cli --raw -h $source_host -p $source_port -n $source_db DUMP "$key" | head -c -1|
redis-cli -x -h $target_host -p $target_port -n $target_db RESTORE "$key" 0; done