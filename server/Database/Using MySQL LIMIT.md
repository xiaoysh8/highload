Using MySQL LIMIT

In Mysql, the LIMIT statement is used to limit the number of results:

SELECT * FROM users ORDER BY id DESC LIMIT 10
# the last 10 entries from the users table

Bias

To return results from 6 to 15, you need to use the following syntax:

SELECT * FROM users ORDER BY id DESC LIMIT 5, 10
# the last 10 entries from the users table, but without the first 5

Or the same:

SELECT * FROM users ORDER BY id DESC LIMIT 10 OFFSET 5
Speed

In terms of performance, LIMIT works very quickly. However, the use of large offset values ​​can lead to performance degradation. For example, the query:

SELECT * FROM users ORDER BY id DESC LIMIT 10
will work much faster than the query:

SELECT * FROM users ORDER BY id DESC LIMIT 9999, 10
In the second case, MySQL will skip 10 thousand lines to show only 10. Read how to optimize large offsets in MySQL .