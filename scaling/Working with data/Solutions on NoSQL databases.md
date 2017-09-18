Solutions on NoSQL databases

In this article, we will consider the principles of solving typical tasks in key-value databases.

Basically, all key-value databases are distributed hash tables . This is their strongest side, because this provides high performance and greatly simplifies scaling. However, this property pops up when you need to work with lists. In MySQL, the following tasks are very simple:

Select 10 latest users
Choose the most popular posts on the blog
Find products with more than 3 comments left
Full text search
Search by any property (find a user whose email is so-so)
But in key-value, these tasks often cause difficulties. Such databases are not at all designed for such tasks. But this is only at first glance, because you can formulate problems in different ways. The decision strategy will depend on the specific situation.

Search by key

There is a class of tasks when you need to do a selection of one object, not by primary key, but by secondary key (for example, user search by email, author search by nickname etc.). The principle of solving this problem is the following: after creating a new object, you need to create references to its primary key for all its properties, for which you will have to make a selection. For example, we create a user:

user_134: {
  name: Den,
  email: golotyuk@gmail.com
}
And duplicate the email key:
user_email_golotyuk@gmail.com: {
  id: 134
}
Now, you will be able to sample the user's data at his mailing address in two stages.

Using RDBMS

If you need MySQL - use it!

There is a class of tasks for which you can and should use RDBMS, such as MySQL and Postgres. If you need to make selections of lists with filters and sorts, then you should use a more suitable RDBMS. A good example is blog posts. (the samples that most likely will need to be made are the newest, most popular, most commented entries, etc.).

The approach is quite simple, but in practice very often the tasks are strongly intertwined, and in the end you can come to the conclusion that RDBMS will take all tasks back on itself. In this case, you should think about a hybrid solution:

Hybrid solution - additional RDBMS

This solution is a mixture of key-value and RDBMS. All the data you store in key-value, but those properties that you need to do aggregate selections are duplicated in RDBMS (with a pointer to the key). Thus. You save the performance of the key-value database and will be able to solve your task. The difficulty here is that you will have to monitor the synchronization of data in both databases, which can greatly complicate the logic of the application.

Full-text search and retrieval with delays

One of the frequent tasks is full-text search. There are excellent solutions - Sphinxsearch, Solr and others. Unfortunately, none of them support key-value database indexing yet, but all provide interfaces for manual indexing. You will only need to describe the implementation for a particular data set (for example, using XmlPipe in Sphinx).

Along with full-text search, there are often tasks related to aggregate samples that are not critical to the current state of the data. For example, when you build a user rating (or posts in a blog, or products in a store or ...), you can safely use the data for the past hour (day / week / ...). As a solution, you can use full-text servers for their indirect purpose. Many of them support a variety of filters and sorting, which is enough to solve 90% of tasks of this kind.

Distributed Samples

Some key-value databases allow you to make selections of lists using built-in tools. The scaling of this solution will look like requests to all nodes of the network and aggregation of results on the backend. On a large scale, this can be a very costly operation, so this solution should be optimized. In many cases, you can do without queries to all nodes, and make a selection on only one of them (for example - select a random photo).