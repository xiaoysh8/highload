Removing large amounts of data from Mysql tables

If you have to delete tens and hundreds of thousands of records from tables, you know that this works slowly. Clearly, after all Mysql in this case should walk through each record and remove it from the disk.

In .io we use (so called) window tables. This is when you store the data in the table in just the last hour (or other interval). So, you not only write a lot there, but also delete a lot. In case the loss of data is not fatal, you can use the MEMORY table. However, for most cases this, of course, will not work. After all, you do not want to lose data every time you reboot the server.

Why DELETE is slow

First of all, it is necessary to understand that the removal of 10,000 records is 10,000 deletions per record. And each deletion is a few operations of recording changes (in data and indexes) on a disk.

In addition, before deleting records, Mysql must first select them. In this case, the same rules are used as for the samples. If the indexes are not configured correctly, the DELETE operation will become even slower.

Setting indexes

To check the use of indexes, it is enough to replace DELETE with SELECT count (*) :

DELETE FROM users WHERE ts < 5
# Let's find out whether the index is used here

EXPLAIN SELECT count(*) FROM users WHERE ts < 5
# Substitution in SELECT will allow you to check the use of indexes

Then we can make sure that the problem is in the index:

+ ---- + ------------- + ------- + ------ + --------------- + ------ + --------- + ------ + ------ + ------------- +
| | id | select_type | table | type | possible_keys | key   | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + ------ + --------------- + ------ + --------- + ------ + ------ + ------------- +
| | 1 | SIMPLE | users | ALL | NULL | NULL | NULL | NULL | 3224 | Using where |
+ ---- + ------------- + ------- + ------ + --------------- + ------ + --------- + ------ + ------ + ------------- +
# It is worth setting the index on the ts column to speed up the deletion

Using Partitions

Indices will help with the removal of relatively small volumes. However, if you have to permanently delete a lot (as in our case), it is worth looking at the partirovanie .

Partitioning allows you to split the table into several physical blocks
This means that we will have the opportunity to manipulate individual partitions. And instead of deleting a large number of records, we can unite them into one block (partition) and delete it with one operation.

To test this in practice, we will create a simple table of such a structure:

tmp : id | title | datetime
# id, headline and date of creation of the title

Fill it with test data (several tens of thousands of records) and delete the data:

DELETE FROM tmp WHERE datetime < '2017-05-31 11:00:00'
# Delete some of the data from the table

The query was executed quite slowly, removing about 25 thousand records:

Query OK, 25750 rows affected ( 0.27 sec )
Make sure that the problem is not in the index (we created it, but just in case we check):

EXPLAIN SELECT count(*) FROM tmp WHERE datetime < '2017-05-31 11:00:00';
We will see that the index is used - everything is fine here:

+ ---- + ------------- + ------- + ------------ + ------- + - -------------- + ---------- + --------- + ------ + ------- + ---------- + -------------------------- +
| | id | select_type | table | partitions | type | possible_keys | key       | key_len | ref | rows | filtered | Extra |
+ ---- + ------------- + ------- + ------------ + ------- + - -------------- + ---------- + --------- + ------ + ------- + ---------- + -------------------------- +
| | 1 | SIMPLE | tmp | NULL | range | datetime | datetime | 5 | NULL | 28395 | 100.00 | Using where; Using index |
+ ---- + ------------- + ------- + ------------ + ------- + - -------------- + ---------- + --------- + ------ + ------- + ---------- + -------------------------- +
Let's also check the system variable, which shows the number of deleted records from all InnoDB tables:

SHOW GLOBAL STATUS LIKE 'Innodb_rows_deleted'\G
Variable_name: Innodb_rows_deleted
        Value: 25750
# See the number of deleted rows

We are experimenting in an isolated environment, so there are no other deletions.

Choice of the partition scheme

Since we are solving the removal problem, we need to have a scheme in which we can conveniently delete (clean) entire partitions. We need to delete the data in an hour, so we will create a HASH partition based on the hour from the datetime field :

ALTER TABLE tmp PARTITION BY hash( HOUR(datetime) ) PARTITIONS 24;
# 24 partitions because 24 hours in a day

Let's check how the distribution of our data on the partitions looks:

SELECT PARTITION_ORDINAL_POSITION, TABLE_ROWS, PARTITION_METHOD
FROM information_schema.PARTITIONS 
WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 'tmp';
+ ---------------------------- + ------------ + ------- ----------- +
| | PARTITION_ORDINAL_POSITION | TABLE_ROWS | PARTITION_METHOD |
+ ---------------------------- + ------------ + ------- ----------- +
| | 1 | 0 | HASH |
| | 2 | 0 | HASH |
| | 3 | 0 | HASH |
| | 4 | 0 | HASH |
| | 5 | 0 | HASH |
| | 6 | 0 | HASH |
| | 7 | 0 | HASH |
| | 8 | 0 | HASH |
| | 9 | 0 | HASH |
| | 10 | 0 | HASH |
| |                         11 | 25394 | HASH |
| |                         12 | 31171 | HASH |
| | 13 | 0 | HASH |
| | 14 | 0 | HASH |
| | 15 | 0 | HASH |
| | 16 | 0 | HASH |
| | 17 | 0 | HASH |
| | 18 | 0 | HASH |
| | 19 | 0 | HASH |
| | 20 | 0 | HASH |
| | 21 | 0 | HASH |
| | 22 | 0 | HASH |
| | 23 | 0 | HASH |
| | 24 | 0 | HASH |
+ ---------------------------- + ------------ + ------- ----------- +
# Partition number will match the hour of the datetime column

As you can see, the data in the table is placed only in two partitions. They correspond to the current and the previous hour. What we need is to clear the party for the hour that we no longer need. For this, there is a TRUNCATE  operation :

ALTER TABLE tmp TRUNCATE PARTITION p11
# This operation was performed for (0.01 sec)

If we check the counter of remote InnoDB records, we'll see there:

Variable_name: Innodb_rows_deleted
        Value: 25750
# The value did not change

This is confirmed by the fact that TRUNCATE does not work principally as DELETE . Instead of deleting each entry, the table (or its partition) is cleared at the structure level. If very rough, then Mysql deletes the old data file and creates a new one. And this operation is performed much faster than the progressive deletion.

TL; DR

If you need to delete large amounts of data from Mysql, follow two tips:

Build indexes to speed up the selection when deleting, replacing DELETE FROM with EXPLAIN SELECT count (*) FROM .
Use partitioning and TRUNCATE PARTITION to effectively remove a large number of rows.