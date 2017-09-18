Selecting Data Types in Mysql

When creating tables in Mysql, we define data types and additional rules for columns (size, indexes, constraints):

CREATE TABLE `users` (

  id int unsigned NOT NULL AUTO_INCREMENT,
  email varchar(128) NOT NULL,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
  gender char(1) NOT NULL,

  PRIMARY KEY (id),
  UNIQUE KEY email (email)

) ENGINE=InnoDB;
# Example of creating a table in Mysql

How to choose the "right" data types? Very simple - you need to comply with the rule "less is better"The less space the values ​​in the table take, the easier it will be for the database to read and write them.

Do you need all the speakers?

First, ask your application a couple of questions. Do you need all the speakers? Maybe just enough for some?

Remove all extra columns
Do not try to guess the future. It is practically guaranteed that you will change the structure of the table with time. Stop only on the data that is needed now.

What is the shortest version of the data?

Should I store the user's gender in full length? Or will it be enough for one letter (f / m)? Is it worth storing the user's phone with the country code, or is it just a direct number?

Reduce the length of all columns to a minimum
Ask these questions for all columns of the future table.

NULL values

The NULL value in Mysql is a special value. To work with it, special functions are provided. For its processing, additional logic is needed. A good rule of thumb is to avoid using this value. Instead, you can use null values ​​for strings or zeros for numbers:


CREATE TABLE `users` (
  ...
  age tinyint NOT NULL DEFAULT 0,
  gender char(1) NOT NULL DEFAULT '',
  ...
);
#NULL value will not be used

However, do not take this as a limitation. In some cases, it is convenient to use NULL to denote the missing value. For example, in the DATETIME columns:

CREATE TABLE `users` (
  ...
  signuped_at datetime NULL DEFAULT NULL,
  ...
);
Whole numbers

For all numeric columns, be sure to calculate the maximum value. There are 4 integer types in Mysql:

TINYINT : 8 bits, maximum 127
SMALLINT : 16 bit, maximum 32,676
INT : 32 bit, maximum 2 x 10 9
BIGINT : 64 bit, maximum 9 x 10 18
Imagine that you are using the INT type for the column in which the user's age is stored. Then, as you are of the type TINYINT , you use 32 - 8 = 24 bits more. For each line. If you have 10 thousand users, you are wasting your time: 24/8 * 10 000 = 30 KB . If users 10 million, then 30 MB .

Select the minimum data type based on the maximum value of the column.
CREATE TABLE `users` (
  ...
  age TINYINT NOT NULL DEFAULT 0,
  ...
);
This may not be so much for the disk, but it is critical for RAM.

UNSIGNED

If a negative number is not relevant for the column, use UNSIGNED values. Then the maximum value will be twice as large, but the minimum will be zero:

UNSIGNED TINYINT : 8 bits, maximum 255
UNSIGNED SMALLINT : 16 bit, maximum 65 535
UNSIGNED INT : 32 bit, maximum 4 x 10 9
UNSIGNED BIGINT : 64 bit, maximum 18 x 10 18
Length of numeric types

In Mysql, you can specify the length of the column after specifying a numeric type:

CREATE TABLE `users` (
  ...
  phone INT(7) NOT NULL DEFAULT 0,
  ...
);
it has no influenceneither by the column size nor by the maximum number. Just never use length for numeric types.

Large numbers

To store very large exact numbers, Mysql suggests using the DECIMAL type :

CREATE TABLE `planets` (
  ...
  distance_to_sun DECIMAL(40,3) NOT NULL DEFAULT 0,
  ...
);
# Use DECIMAL for a non-integer number

In parentheses, the number of digits of the total and their number after the decimal point (may be zero) are indicated. Since the processors do not support mathematical operations with such numbers, Mysql does all the calculations on its side. Therefore,it's very slow.

FLOAT / DOUBLE

Unlike DECIMAL , the FLOAT type is approximate (it stores an inaccurate number). At the same time, the processor is able to work with this type directly. In addition, FLOAT takes up less space than DECIMAL for storing the same values.

Use FLOAT / DOUBLE instead of DECIMAL if you do not need very accurate numbers
VARCHAR / CHAR

When you select row types, the minimum rule also applies. Estimate the maximum length of the string and set the constraint. Type CHAR is a fixed length type. This means that for any line, the same number of bytes will always be allocated:

CREATE TABLE `planets` (
  ...
  state CHAR(2) NOT NULL DEFAULT '',
  ...
);
# the column will always occupy 2 bytes (even if it is empty)

VARCHAR is a variable-length type. In such a column, the string will occupy exactly its length (in bytes):

CREATE TABLE `planets` (
  ...
  first_name VARCHAR(32) NOT NULL DEFAULT '',
  ...
);
# the column will occupy from 1 to 32 bytes depending on the specific value

However, Mysql will add 1 more bytes to store the length of the string itself. Also it is worth considering that updating such a line can be an expensive operation (fraught with fragmentation of data, which means slowing down the reading). Use this rule:

If the values ​​in the text column are similar in length, select CHAR, otherwise - VARCHAR.
BLOB / TEXT

The TEXT and BLOB types differ only in that for the second type Mysql does not convert the encodings (stores as is).

Do not use TEXT / BLOB types for sorting columns
Mysql does not know how to sort by these values, so it uses only the first max_sort_length characters. Similarly, when creating an index for this column, you need to specify the length:

CREATE INDEX article_body ON articles ( body(32) );
# Specify the length of the column for indexing

It's hard to imagine why you need to index text columns, just do not do it. If you want to use the ability to create an index for checking uniqueness, use the auxiliary column to store the md5 hash from the text:

CREATE TABLE `articles` (
  body TEXT NOT NULL DEFAULT '',
  body_md5 CHAR(32)
);
# You can create a unique index on the body_md5 column

ENUM

The row values ​​can be one of the known values ​​(for example, the list of countries). If the list is fixed (new countries do not appear every day), it will be convenient to use the ENUM type . It allows you to specify a specific list of values ​​and use them only in the rows of the table.

CREATE TABLE users(
...
	region ENUM('EU', 'USA', 'ASIA') NOT NULL
...
);
The advantage of this type is that it writes the value number instead of the value itself in each line. This provides a huge space saving.

Do not use this type for dynamic values. If there is a list of values ​​that can be expanded (for example, product categories), use a separate table with the identifier:

CREATE TABLE posts(
...
	category_id TINYINT UNSIGNED NOT NULL DEFAULT 0,
...
);

CREATE TABLE categories(
	id TINYINT UNSIGNED NOT NULL AUTO_INCREMENT,
	title char(24) NOT NULL
);
# Do not use ENUM for dynamic values

DATETIME / TIMESTAMP

Both date formats allow you to store date and time values ​​up to seconds. However, there are differences between them:

DATETIME takes 8 bytes and allows you to store dates from 1001 to 9999.
TIMESTAMP takes 4 bytes and allows you to store dates from 1970 to 2038.
Use the TIMESTAMP format to set the event dates (for which it was created). For example, the time of registration of a user or the publication of a comment. In addition, it has a convenient mechanism for initializing and updating:

CREATE TABLE users (
...
	sign_up_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	last_visit_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
...
);
# Initializing and auto-updating the TIMESTAMP columns

In other cases, use DATETIME .

TL; DR version

Remove unnecessary columns from the circuit.
Reduce the length of the columns to a minimum.
Avoid using NULL values.
Select the minimum required numeric types ( TINYINT / SMALLINT instead of INT ).
Use FLOAT / DOUBLE instead of DECIMAL for approximate numbers.
Select CHAR for lines of approximately the same length.
For the rest of the lines, select VARCHAR .
Do not use TEXT / BLOB for sorting and indexing.
Use ENUM instead of rows from a fixed set (for example, a list of countries).
Use TIMESTAMP to set the time of events (registration, sending a message, etc.).
For other dates, use DATETIME .
Read about the device indexes and tuning settings in Mysql .