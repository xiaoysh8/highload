Denormalization of data

The normal form of data storage implies avoiding duplication of data. The key rules are two:

Atomicity means that all entities are stored in an indivisible form. For example, if we store the address, it will most likely be divided into the name of the city, the country and the street. All of them must be presented in separate tables. The name of the city will be atomic, because further will not be divided.
Uniqueness requires that each entity be defined only once. For example, the city name with ID 1 must be present only in the cities table.

Normalization ensures the convenience of working with data. So, when updating the city name, you will need to make a change in only one entry in the city table.

But in terms of performance, normalization is very expensive. To select the name of the user's city, we will need to make several requests instead of one. JOINs have a negative impact on application performance.

Denormalization

Denormalization is a gradual process of getting rid of the rules of normalization where necessary. Usually these are cases in which there are frequent repeated requests to logically related data. For example, the constant selection of user data and the name of his city from two tables.

There are two main approaches for denormalizing data:

Duplication.
Preliminary preparation.
1. Data duplication

Suppose we have tables of this structure:

users
  id
  name
  city_id
cities
  id
  title
  country
+         +
| users   |
+         +
| id	  |
| name	  |
| city_id |
+         +

+         +
| cities  |
+         +
| id      |
| title   |
| country |
+         +
Inserting a new user will look like this:

<?
$city_id = 15;
mysql_query('INSERT INTO users SET name = "' . $name . '", city_id = ' . $city_id);
To sample the name of the city or country of the user, we need to make two queries or one JOIN:

SELECT * FROM users u JOIN cities c ON (c.id = u.city_id)

In order to take advantage of the duplication, we need to add the column city_title to the users table:

users
  id
  name
  city_id
  city_title
When inserting the user, this column will need to save the city name. Now inserting a new user will look like this:

<?
$city_id = 15;
$city = mysql_fetch_assoc( mysql_query('SELECT * FROM cities WHERE id = ' . $city_id) );
mysql_query('INSERT INTO users SET name = "' . $name . '", city_id = ' . $city_id . ', city_title="' . $city['title'] . '"');
As a result, we can select the user's data immediately with the name of the city for one simple query:

SELECT id, name, city_title FROM users
One-to-many relationships

One-to-many relationships can also be optimized using duplication. For example, let's imagine a table of posts marked with a blog:

posts
  id
  title
  body
tags
  id
  title
  
post_tags
  post_id
  tag_id
  
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
+            +  
| post_tags  |
+            +
| post_id    |
| tag_id     |
+            +
To sample post labels, we need to make two separate requests (or one JOIN):

SELECT * FROM tags t JOIN post_tags pt ON (pt.tag_id = t.id) WHERE pt.post_id = 1;
Instead, we could save the list of labels in a separate text column separated by commas:

posts
  id
  title
  body
  tags
Then at the conclusion of posts it will not be necessary to make additional inquiries to obtain a list of labels.

2. Preliminary preparation of data

Aggregate queries are usually the heaviest. For example, getting the number of records for a specific condition:

SELECT count(*) FROM users WHERE group_id = 17
In addition to duplicating data from one table to another, you can also save data that is calculated. Then it will be possible to avoid heavy aggregate samples. 

For example, to store the number of users in a group, you need to add an additional column:

groups
  id
  title
  user_count
Then, at each addition of the user, it will be necessary to increase the value in the column user_count by 1:

UPDATE groups SET user_count = user_count + 1 WHERE id = 17
Such a data storage scheme is usually called facts + measurements: 

In this case, we have tables with basic data (facts) and measurement tables, where the calculated data are stored.

Vertical tables

The vertical structure uses the rows of the table to store the name of the fields and their values. 

The advantage of storing data in this form is the possibility of convenient shading . In addition, it will be possible to add new fields without changing the structure of the table. Vertical structure is well suited for tables in which columns can vary.

A similar structure and advantages are possessed by the Key-Value database .

The most important

Denormalization is important to implement gradually and only for those cases when there are repeated samples of related data from different tables. Remember, doubling the data will increase the number of records, but the number of readings will decrease. The calculated data is also conveniently stored in columns to avoid unnecessary aggregate samples.