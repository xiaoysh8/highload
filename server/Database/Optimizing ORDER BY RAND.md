Optimizing ORDER BY RAND ()

How to choose a random entry from a table in Mysql?

SELECT id FROM files ORDER BY rand() LIMIT 1;
But such requests are very slow. Let's take a look at EXPLAIN :

EXPLAIN SELECT id FROM files ORDER BY rand() LIMIT 1;
We will see that Mysql creates a temporary table and uses the sorting of all data. Such a query will work more slowly when the table is full:

+ ---- + ----- + ------ + ------------------------------- --------------- +
| | id | ... | rows | Extra |
+ ---- + ----- + ------ + ------------------------------- --------------- +
| | 1 | ... | 4921 | Using index; Using temporary; Using filesort |
+ ---- + ----- + ------ + ------------------------------- --------------- +
The right decisionwill use the index and get rid of ORDER BY RAND () . For this you need:

Determine the maximum value of the ID (yes, such a column must be and it must be a key ) in the table.
Get any random number from zero to the maximum ID.
Select the first entry from the table where the ID is greater than the specified random number, sorting it by the same column.
If you translate everything into a query:

SELECT f.id FROM files f
	    JOIN ( SELECT RAND() * (SELECT MAX(id) FROM files) AS max_id ) AS m
	    WHERE f.id >= m.max_id
	    ORDER BY f.id ASC
	    LIMIT 1;
# Efficient substitution ORDER BY RAND ()

How it works

In the nested querywe determine the maximum value of ID. Suppose it is 100,000.
Then we multiply this value by the function RAND (). It returns a value from 0 to 1. Let the example be 0.5. Then the multiplication result is 50,000.
After that, this value with JOIN is added to each line of the original table.
Filter f.id> = m.max_id selects the first record that will be found, the ID of which will be more than 50,000.
Because we used sorting ORDER BY f.id ASC, all skipped entries will be less than 50,000.
This means that we selected a random entry from the entire table. But unlike ORDER BY RAND () , we used sorting and filtering by the ID index (and therefore effectively).
The speed of such a query will be several times faster than the original one:

mysql> SELECT id FROM files ORDER BY rand () LIMIT 1;
+ ------- +
| | id |
+ ------- +
| | 72643 |
+ ------- +
1 row in set (0.17 sec)
Accelerated version:

mysql> SELECT f.id FROM files f JOIN (SELECT rand () * (SELECT max (id) from files) AS max_id) AS m WHERE f.id> = m.max_id ORDER BY f.id ASC LIMIT 1;
+ ------- +
| | id |
+ ------- +
| | 86949 |
+ ------- +
1 row in set (0.00 sec)
Now it works quickly and does not depend on the size of the table.