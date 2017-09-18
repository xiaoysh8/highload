5 strategies for working with high loads in MySQL

MySQL is a proven and very powerful technology. Including for building systems with heavy load. Even Facebook uses Mysql to manage huge amounts of data. Let's consider the basic strategies for building loaded systems based on MySQL.

Optimization and indexes

First of all, make sure that you use all the standard features of the database. Correct work with indexes will give huge gains in productivity. Hundreds and even thousands of times. While freeing up resources for other tasks. Today, the cost of hard drives is constantly decreasing, and the requirements for speed are constantly growing.

Indexes - is an effective mechanism to transfer the load from the processor to the hard disk in the correct proportions.
Do not rush to optimize all requests in advance. Use the slow query log to understand where there are real problems.

Immediately after installing MySQL, do not forget to optimize the basic parameters . The standard setting is very basic and is focused on modest hardware and strict security requirements.

Adjustment of the standard parameters will give a significant increase in the acceleration of not only read but also write operations.
Caching

Caching is a very popular method of performance optimization.

Internal Mysql cache

Before you use an external solution, think about whether to use the internal Mysql cache. It makes sense to include it in those cases when Mysql works with a very large number of readings ( SELECT ), but not very large (at least 10 times less) records ( INSERT , DELETE and UPDATE ).

It is better not to include the internal Mysql cache in environments with a large number of entries / updates.
The cache is configured using the mysql_query_cache_size parameter .

External solutions

A more flexible solution is to use external caching tools, like Memcache or Redis . There are a number of techniques for caching data in applications.

However, be careful. Caching is often not a solution to a problem, but rather a postponement. Slow query becomes even slower, and its impact (when resetting the cache) is less than predicted.

Caching is best used only as an intermediate solution. In the end, you should get rid of slow requests.
Replication

Despite the fact that replication can help to cope with the load, it is better not to use it for this. It should be remembered that along with scaling, you will always have the issue of accessibility. If a replica that helps service requests fails, what will happen to the system?

On the other hand, the replica just allows you to ensure high availability. One approach looks like this:

Use master-slave replication for each database server.
The application always works only with the wizard.
If the wizard fails, the application switches to the slave.
We at this time raise the broken server and turn it into a slave ( how to do it right ).
Thus, in the new scheme, the master and slave were interchanged, and the application (ie its users) did not notice any problems.

Replication should only be used as a backup mechanism. Not for scaling under load.
Sharding

Sharding is the principle of scaling a database when data is shared across different servers. We have at our disposal two approaches:

Vertical Sharding

It should be used first. This is a simple distribution of tables across servers. For example, you put the users table on one server, and the orders table on the other. In this case, the groups of tables for which JOIN is executed must be on the same server.

Horizontal Sharding

This type of shading should be used in the next step. At this stage, very large tables that cease to fit on one server are divided into parts and placed different parts of them on different servers. This complicates the logic of the application, but the world has not yet come up with better scaling mechanisms.

Sharding is the only approach for scaling really large data.
Other tasks

It should be noted that there are tasks with which MySQL is doing extremely poorly. One example is the sampling of unique values ​​in different ranges . Or full-text search .

Pay attention to Handlersocket , which can become a replacement for any NoSQL solution, if it is used for simple Key-Value operations.

MySQL is a powerful, but not a universal solution. Redis, Elastic and other technologies will help to solve additional problems.
The most important

MySQL along with modern approaches to optimization and scaling is a powerful platform for building huge systems. Do not forget to use other technologies for related tasks, which MySQL can not handle so effectively.