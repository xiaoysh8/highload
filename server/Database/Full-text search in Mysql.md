Full-text search in Mysql

A simple text search in Mysql can be implemented using the LIKE operator:

SELECT * FROM articles WHERE title LIKE '%database%'
# Simple search for occurrences in the text

However, this type of search crashes when the search query becomes a little more complicated:

SELECT * FROM articles WHERE title LIKE '%configure database%'
# Text will not be found if it occurs, for example, "configure your database"

For more advanced search, there are special full-text indexes (or full-text search indexes). Mysql supports full-text indexes for MyISAM tables. The Innodb support is added from version 5.6.4.

To use full-text search, you need to declare it when creating the table:

CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT,
      FULLTEXT (title,body)
    ) ENGINE=MyISAM;
# Innodb is supported since version 5.6.4

Alternatively, you can add an index to an existing table:

CREATE FULLTEXT INDEX title_body ON articles(title,body)
To search, it is sufficient to use the MATCH operator:

SELECT * FROM articles WHERE MATCH (title,body) AGAINST ('configured mysql');
# Mysql will search for matches in two columns at once