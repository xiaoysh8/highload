Data denormalization

The normal form of storage involves avoiding data duplication. There are two key rules:

Atomicity means that all entities are stored in the undivided state. For example, if we store the address, it is likely to be divided into the country, the city, and the street. All of them should be presented in separate tables. The town's name is atomic because it can't be divided.
Uniqueness requires that each substance was determined only once. For example, the city with the identifier 1 should be present only in the cities table.

Normalization provides the convenience of working with data. For example, when upgrading the town's name, you will need to make a change in one record of the table.

But in terms of performance normalization is very expensive. To select the user's city, we need to make couple queries instead of one. JOIN's have a negative impact on application's performance.

Denormalization

Denormalization is a gradual process of getting rid of normalization rules where necessary. Typically, in cases with frequently repeated requests to the logically related data. For example, the constant query of the user data and the name of his city from two tables.

There are two main approaches in the denormalization of data:

Duplication;
Preliminary preparation.
1. Data duplication

Let's assume we have tables with similar structure:

+         +
| users   |
+         +
| id    |
| name    |
| city_id |
+         +

+         +
| cities  |
+         +
| id      |
| title   |
| country |
+         +
Inserting a new user would look like this:

<?
$city_id = 15;
mysql_query('INSERT INTO users SET name = "' . $name . '", city_id = ' . $city_id);
To retrieve the name of the city or country of the user, we need to make two requests or one JOIN:

SELECT * FROM users u JOIN cities c ON (c.id = u.city_id)

In order to use the duplicate advantage, we need to add a column city_title to the users table:

+           +
| users     |
+           +
| id      |
| name      |
| city_id   |
| city_title|
+           +
Inserting a user into the column you will need to save the city name. Now insert of the new user will look like this:

<?
$city_id = 15;
$city = mysql_fetch_assoc( mysql_query('SELECT * FROM cities WHERE id = ' . $city_id) );
mysql_query('INSERT INTO users SET name = "' . $name . '", city_id = ' . $city_id . ', city_title="' . $city['title'] . '"');
As a result, we can choose the user data with the name of the city in one simple query:

SELECT id, name, city_title FROM users
Ties one to many

Ties one-to-many can be optimized using the duplication. Assume we have a set of tables of tagged posts entries:

+         +
|  posts  |
+         +
| id      |
| title   |
| body    |
+         +  
+         +
|  tags   |
+         +
|  id     |
|  title  |
+         +
+           +  
| post_tags |
+           +
| post_id   |
| tag_id    |
+           +
For tags queries, we need to make two separate queries (or one JOIN):

SELECT * FROM tags t JOIN post_tags pt ON (pt.tag_id = t.id) WHERE pt.post_id = 1;
Instead, we could save a list of tags in a single column of text separated by commas:

+        +
| posts  |
+        +
| id     |
| title  |
| body   |
| tags   |
+        +
Then for the derivation of posts, additional requests for the list of tags won't be necessary.

2. Preliminary data preparation

Aggregate queries are usually the most heavy. For example, to obtain the number of records for a particular condition:

SELECT count(*) FROM users WHERE group_id = 17
In addition to duplicating data from one table to another, you can also store calculated data. Then it will be possible to avoid the heavy aggregated queries. 

For example, it is necessary to add an extra column to store the number of users in the group:

+             +
|  tags       |
+             +
|  id         |
|  title      |
|  user_count |
+             +
Then, every time a user is added, it will be necessary to increase the value in column user_count by 1:

UPDATE groups SET user_count = user_count + 1 WHERE id = 17
Such data storage scheme is commonly called facts + measurements: 

At the same time we have tables with the main data (facts) and the measurements tables where the calculated data is stored.

Vertical tables

The vertical structure uses a string table to store the names of the fields and their values. 

The advantage of storing data in this way is a convenient opportunity for sharding. In addition, it will be able to add new fields without changing the structure of the table. The vertical structure is well suited for tables, which may need changes in the columns.

Similar structures and advantages have Key-Value databases.

The most important

It is important to implement denormalization gradually and only in those cases when there are repeated sampling of related data from different tables. Remember, when you duplicate the data the number of entries will also grow, but the number of readings will decrease. Calculated data is also convenient to store in a column in order to avoid unnecessary aggregate queries.