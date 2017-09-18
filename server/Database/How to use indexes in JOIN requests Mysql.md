How to use indexes in JOIN requests Mysql

Working with indexes in Mysql is a fundamental task for building systems with high performance. In this article, let's look at how Mysql uses indexes in JOIN queries.

Introductory

Imagine that we have a simple system for counting the statistics of the articles viewed. Data on articles we store in one table:

+ ------- + -------------- + ------ + ----- + ------------- ------ + ----------------------------- +
| | Field | Type | Null | Key | Default | Extra |
+ ------- + -------------- + ------ + ----- + ------------- ------ + ----------------------------- +
| | id     | int (11) | NO | PRI | 0 | | |
| | ts | timestamp | NO | | | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| | title | varchar (512) | YES | | | NULL | | |
+ ------- + -------------- + ------ + ----- + ------------- ------ + ----------------------------- +
Data with statistics are stored in another table with this structure:

+ ------------ + --------- + ------ + ----- + ------------ + ---------------- +
| | Field | Type | Null | Key | Default | Extra |
+ ------------ + --------- + ------ + ----- + ------------ + ---------------- +
| | url_id | int (11) | NO | PRI | NULL | auto_increment |
| | article_id | int (11) | NO | | | 0 | | |
| | date | date | NO | | | 0000-00-00 | | |
| | pageviews | int (11) | YES | | | NULL | | |
| | uniques | int (11) | YES | | | NULL | | |
+ ------------ + --------- + ------ + ----- + ------------ + ---------------- +
Note that in the second table, the primary key is url_id . This is the link reference for the article. Those. one article can have several different links, and for each of them we will collect statistics. The column article_id corresponds to the column id from the first table. The statistics itself is very simple - the number of views and unique visitors per day.

Selection of statistics for one article

Let's make a choice of statistics for one article:

SELECT s.article_id, s.date, SUM(s.pageviews), SUM(s.uniques)
FROM articles a
JOIN articles_stats s ON (s.article_id = a.id)
WHERE a.id = 4
GROUP BY s.date;
# Statistics for articles with id = 4

At the output we get views and unique visitors for this article for each day:

+ ------------ + ------------ + ------------------ + ---- ------------ +
| | article_id | date | SUM (s.pageviews) | SUM (s.uniques) |
+ ------------ + ------------ + ------------------ + ---- ------------ +
| | 4 | 2016-01-03 | 28920 | 9640 |
... ...
| | 4 | 2016-01-07 | 1765 | 441 |
+ ------------ + ------------ + ------------------ + ---- ------------ +
499 rows in set (0.37 sec)
The query worked for 0.37 seconds, which is quite slow. Let's take a look at EXPLAIN :

+ ---- + ------------- + ------- + ------- + -------------- - + --------- + --------- + ------- + -------- + ----------- ----------------------------------- +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + ------- + -------------- - + --------- + --------- + ------- + -------- + ----------- ----------------------------------- +
| | 1 | SIMPLE | a | const | PRIMARY | PRIMARY | 4 | const |      1 | Using index; Using temporary; Using filesort |
| | 1 | SIMPLE | s | ALL | NULL | NULL     | NULL | NULL | 676786 | Using where                                   |
+ ---- + ------------- + ------- + ------- + -------------- - + --------- + --------- + ------- + -------- + ----------- ----------------------------------- +
EXPLAIN shows two records - one for each table from our query:

For the first table, Mysql chose the PRIMARY index and effectively used it.
For the second table, Mysql could not find suitable indexes and had to check almost 700 thousand records to generate the result.
In JOIN queries, Mysql will use an index that will filter most of the records from one of the tables
Therefore, we need to make sure that Mysql will quickly execute the query of this type:

SELECT article_id, date, SUM(pageviews), SUM(uniques) FROM articles_stats WHERE article_id = 4 GROUP BY date
According to the logic of index selection, we build an index on the column article_id :

CREATE INDEX article_id on articles_stats(article_id);
Let's check the use of indexes in our first query:

EXPLAIN SELECT s.article_id, s.date, SUM(s.pageviews), SUM(s.uniques) from articles a join articles_stats s on (s.article_id = a.id) where a.id = 4 group by s.date;
And we will see that Mysql now uses indexes for two tables:

+ ---- + ------------- + ------- + ------- + -------------- - + ------------ + --------- + ------- + ------ + ---------- ------------------------------------ +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + ------- + -------------- - + ------------ + --------- + ------- + ------ + ---------- ------------------------------------ +
| | 1 | SIMPLE | a | const | PRIMARY | PRIMARY     | 4 | const |    1 | Using index; Using temporary; Using filesort |
| | 1 | SIMPLE | s | ref | article_id | article_id | 4 | const |  677 | Using where |
+ ---- + ------------- + ------- + ------- + -------------- - + ------------ + --------- + ------- + ------ + ---------- ------------------------------------ +
This will significantly speed up the request (after all, Mysql in the second case processes 1000 times less data).

Aggregation requests

The previous example is more laboratory. A query more approximate to practice is the sampling of statistics on several articles:

SELECT s.article_id, s.date, SUM(s.pageviews), SUM(s.uniques), a.title, a.ts
FROM articles a
JOIN articles_stats s ON (s.article_id = a.id)
WHERE a.id IN (4,5,6,7)
GROUP BY s.date;
However, in this case Mysql will behave exactly the same way. It will evaluate which indexes can be used from each table. EXPLAIN will show:

+ ---- + ------------- + ------- + -------- + ------------- - + ------------ + --------- + ------------------- + ---- - + ---------------------------------------------- +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + -------- + ------------- - + ------------ + --------- + ------------------- + ---- - + ---------------------------------------------- +
| | 1 | SIMPLE | s | range | article_id | article_id | 4 | NULL | 2030 | Using where; Using temporary; Using filesort |
| | 1 | SIMPLE | a | eq_ref | PRIMARY | PRIMARY     | 4 | test.s.article_id |    1 | Using index |
+ ---- + ------------- + ------- + -------- + ------------- - + ------------ + --------- + ------------------- + ---- - + ---------------------------------------------- +
The tables will be processed in a different order. First, all suitable values ​​from the statistics table will be sampled. And then from the table with the names.

Mysql decided that by first selecting the statistics for all the necessary articles, it would then make a quick selection from the articles table . The order in this case does not really matter, because in the articles table the selection is based on the primary key.

Additional filters

In practice, you have to deal with additional filters in queries. For example, a sample of statistics only for a certain date:

SELECT s.article_id, s.date, SUM(s.pageviews), SUM(s.uniques), a.title, a.ts
FROM articles a
JOIN articles_stats s ON (s.article_id = a.id)
WHERE s.date = '2017-05-14'
GROUP BY article_id
In this case, Mysql will not be able to find the index for the statistics table again:

+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- - + ------------- +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- - + ------------- +
| | 1 | SIMPLE | s | ALL | article_id | NULL     | NULL | NULL | 676786 | Using where |
| | 1 | SIMPLE | a | eq_ref | PRIMARY | PRIMARY | 4 | test.s.article_id | 1 | | |
+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- - + ------------- +
The logic for choosing an index here is the same as in the previous example. It is necessary to choose an index that will allow you to quickly filter the statistics table by date:

CREATE INDEX date ON articles_stats(date);
Now the query will use indexes on both tables:

+ ---- + ------------- + ------- + -------- + ------------- ---- + --------- + --------- + ------------------- + ----- - + ---------------------------------------------- +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + -------- + ------------- ---- + --------- + --------- + ------------------- + ----- - + ---------------------------------------------- +
| | 1 | SIMPLE | s | ref | article_id, date | date     | 4 | const | 2996 | Using where; Using temporary; Using filesort |
| | 1 | SIMPLE | a | eq_ref | PRIMARY | PRIMARY | 4 | test.s.article_id | 1 | | |
+ ---- + ------------- + ------- + -------- + ------------- ---- + --------- + --------- + ------------------- + ----- - + ---------------------------------------------- +
Complex filters and sorting

In even more complex cases, the samples include additional filters or sorting. Let's say we want to select all articles created not later than a month ago. And show statistics for them only for the last day. Only for those publications that have more than 15 thousand unique visits. And the result is sorted by views:

SELECT s.article_id, s.date, SUM(s.pageviews), SUM(s.uniques), a.title, a.ts
FROM articles a
JOIN articles_stats s ON (s.article_id = a.id)
WHERE a.ts > '2017-04-15' AND s.date = '2017-05-14' AND s.uniques > 15000
GROUP BY article_id
ORDER BY s.pageviews
# The query will work for 0.15 seconds, which is quite slow

Mysql will look for indexes that will allow you to filter out the maximum values ​​from each source table. In our case this will be:

+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- + ---------------------------------------------- +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- + ---------------------------------------------- +
| | 1 | SIMPLE | s | range | date | date     | 4 | NULL | 26384 | Using where; Using temporary; Using filesort |
| | 1 | SIMPLE | a | eq_ref | PRIMARY | PRIMARY | 4 | test.s.article_id | 1 | Using where |
+ ---- + ------------- + ------- + -------- + ------------- - + --------- + --------- + ------------------- + ------- + ---------------------------------------------- +
The index date will allow you to filter the statistics table to 26 thousand records. Each of which will have to be checked for compliance with other conditions (the number of unique visitors is more than 15 thousand).

Sorting by views Mysql will in any case do it yourself. Indices here will not help, tk. sort the dynamic values ​​(the result of the GROUP BY operation ).

Therefore, our task is to select an index that will make it possible to reduce the sample by the table of articles_stats using the filter s.date = '2017-05-14' AND s.uniques> 15000 .

Create an index on both columns from the first item:

CREATE INDEX date_uniques ON articles_stats(date,uniques);
Then Mysql can use this index to filter the statistics table:

+ ---- + ------------- + ------- + -------- + ------------- - + -------------- + --------- + ------------------- + - ---- + --------------------------------------------- - +
| | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+ ---- + ------------- + ------- + -------- + ------------- - + -------------- + --------- + ------------------- + - ---- + --------------------------------------------- - +
| | 1 | SIMPLE | s | range | date_uniques | date_uniques | 9        | NULL | 1681 | Using where; Using temporary; Using filesort |
| | 1 | SIMPLE | a | eq_ref | PRIMARY, ts_id | PRIMARY | 4 | test.s.article_id | 1 | Using where |
+ ---- + ------------- + ------- + -------- + ------------- - + -------------- + --------- + ------------------- + - ---- + --------------------------------------------- - +
# With this index, Mysql will process 10 times less records for the sample

Other acceleration strategies JOIN

In situations where it is not possible to select an appropriate index, think about denormalization . In this case, you should use the rule:

Better a lot of easy records than a lot of hard reads
You should create a table that is optimized for the query and synchronously update it. However, make sure that your server is well set up . As an urgent measure, consider the possibility of caching the results of heavy queries.

TL; DR

Use the same rules for creating indexes as for normal queries - the index for columns in the WHERE clause .
In complex samples, select indexes (brute force), which will reduce the number of "rows" in the EXPLAIN query.
In particularly difficult cases, denormalization and caching .