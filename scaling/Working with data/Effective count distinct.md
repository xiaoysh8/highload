Effective count distinct

The choice of the number of unique values ​​for a certain period of time is not a very frequent task. In .io, we need to determine the number of unique users visiting the site for any period of time.

Of course, there is a very simple solution based on normal SQL. It is enough to save in a simple table the unique id of users and the time of their entry:

id | time
Then the solution looks like this:

SELECT count(distinct(id)) FROM visits WHERE time > :start AND time < :end
# Regular sample count distinct in Mysql

However, this solution works quite slowly even on small tables (millions of records).

mysql> SELECT count (distinct (id)), count (*) FROM table WHERE time> date_sub (now (), interval 5 day);
+ --------------------- + ---------- +
| | count (distinct (id)) | count (*) |
+ --------------------- + ---------- +
| | 62501 | 2551500 |
+ --------------------- + ---------- +
1 row in set ( 1.76 sec )
# Very slow, and this is 2.5 million records in the table

The total number of the audience, which we think is about 50 million people a day, that's why it did not suit us.

MySQL and count (distinct)

Mysql is not very suitable for solving problems. But it was worth a try. To make everything work faster, you can make several optimizations.

1. Correct index

Mysql will still scan all the rows from the query to determine the unique id. The time column will help to significantly reduce them. Therefore, it should go first in the index:

CREATE INDEX time_id ON visits(time, id)
2. Quantum of time

In our case, the minimum time interval for calculating the values ​​can not be less than a day. It makes no sense to save more than one record per user per day. So the time column needs to be done like date and put a unique index :

CREATE UNIQUE INDEX time_id ON visits(time, id)
# The column time is of type date, now every day there will be only one record with a unique id

Vertica and count (distinct)

This vector database is well optimized for aggregate samples. However, counting the quantity for the selected time period works only 2-3 times faster than Mysql.

SELECT count (distinct (id)) FROM table WHERE time> now () - interval '5 day';
 count 
-------
 60700
(1 row)

Time: First fetch (1 row): 659,064 ms . All rows formatted: 659,192 ms
# Slightly faster than Mysql

approximate_count_distinct ()

Vertica supports the aggregate function approximate_count_distinct () , which counts the number of unique values ​​approximately. It works an order of magnitude faster than the usual query, but has an error of about 1%:

SELECT approximate_count_distinct(id) FROM table WHERE time > now() - interval '5 day';
Redis and HyperLogLog

Redis has a special HyperLogLog store . It allows you to store keys there, and then get the number of unique keys in this store. The limitation is that you can not get the list of saved keys. The advantage is that one such storage takes only 12 KB, can store 2 64 elements and returns the result with an error of only 0.8% .

HyperLogLog Device

HyperLogLog is a probabilistic algorithm for counting unique elements.

The principle of the algorithm can be explained by a simple example from life. Imagine that you for a while tossed a coin, counting the number of continuous successive fallouts tails. If you get a sequence of just 3 dashes in a row, then you can assume that you tossed a coin not very long. On the other hand, if you get 15, then you probably spent most of the day.

But if you're lucky, and the first time you got a sequence of 10 bars, after which you stopped this senseless occupation, then the approximate judgment of the time spent will be extremely incorrect. To increase the accuracy of the approximation, you will need to repeat the expression, but this time use 10 coins and 10 sheets of paper, on which you will record the longest sequences of fallen decks. Thus, judging about the time spent will be much more accurate.

HyperLogLog calculates the hash of each new item. Part of this hash (in binary representation) is used for the register index (we divide the set of zeros and ones by the m-number of subsets, our pairs of coins + sheets of paper). And the other part is used to calculate the length of the sequence of the first zeros and the maximum value of this sequence (the length of the sequence of fallen decks). The probability of a sequence of n + 1 zeros is half the probability of a sequence of length n.HyperLogLog sheme

Therefore, using the values ​​of different registers that are tied to the maximum sequences of zeros for a given subset, HyperLogLog is able to provide the approximate power of the set with high accuracy. If there are m-subgroups and n-elements, then in each group there will be on average n / m unique elements, and the average for all subgroups gives a fairly accurate estimate of the value of log 2 ( n / m ).

Counting unique values

In the simplest case, it's enough to store all the values ​​in the HLL element:

<?
$r = new Redis;
$r->connect('127.0.0.1', 6379);

$r->pfadd('hll', [1]);
echo $r->pfcount('hll');
# The pfcount function returns the number of unique values ​​in the hll clue, see 1

Here we saved 1 item (number 1) in the "hll" key and deduced the number of unique elements. Add a couple more items to the same key:

<?
$r = new Redis;
$r->connect('127.0.0.1', 6379);

$r->pfadd('hll', [1, 2, 3, 1, 1, 2]);
echo $r->pfcount('hll');
# Now we will see on the screen 3

As it should, we got the result 3 (that is, only 3 unique elements are written in this key).

And most importantly - this approach works 3-4 times faster than a similar solution based on SQL.

Time Filters

And yet, the standard solution is not enough. We need to be able to choose a period for counting unique values. In this case, Redis knows how to glue HLL keys right at the time of sampling:

<?
$r = new Redis;
$r->connect('127.0.0.1', 6379);

$r->pfadd('hll1', [1, 2, 3]);
$r->pfadd('hll2', [1, 4, 5]);

echo $r->pfcount(['hll1', 'hll2']);
# Selection will return 5 - the number of unique elements in two HLL keys

For sampling, you can use any number of hll keys. And we need to get the number of unique users by days. Therefore, for the solution, it is sufficient to store the user ID in the HLL key when visiting the page. The key name will contain the suffix - visit date:

<?
$r = new Redis;
$r->connect('127.0.0.1', 6379);

$id = $_REQUEST['id']; # id посетителя (уникальное для каждого человека)
$key = 'visits' . date('Ymd'); # HLL ключ с суффиксом даты

$r->pfadd($key, [$id]);
# Saving information about a visit per day

Now you need to select the number of unique users for any period of time. To do this, you need to glue all the keys for the dates inside this gap:

<?
$r = new Redis;
$r->connect('127.0.0.1', 6379);

$dates = ['20160610', '20160611', '20160612', '20160613']; # весь промежуток с датами в формате Ymd

foreach ( $dates as $date ) $keys[] = 'visits' . $date;

echo $r->pfcount($keys);
# The result will be the number of unique visitors in the interval from 10.06 to 13.06 2016

The most important thing

SQL tools can not cope with the calculation of unique values. Especially if you need to filter the list. On small amounts of data, it is sufficient to use the correct indexes in MySQL. On volumes of more than 1 million records, you should consider the possibility of using the HyperLogLog store in Redis.