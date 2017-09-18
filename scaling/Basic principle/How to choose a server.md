How to choose a server

Sometimes it's better to buy a new server than to optimize the application. The time of developers now costs much more than servers. How to choose a server for growth and new tasks?

The key nodes of any application are backend, frontend and database.

Backend


On the backends, there is usually a server application (PHP, Python, Ruby, etc.). Almost always, the application is demanding for processor power. Source codes usually take up very little space, so the RAM is not significant.

The load on the disk will be very low (reading files when updating the code). Considering that backends are usually easily replaced, it makes no sense to put RAID arrays, only one disk is enough. However, you need to have at least two backends.

Therefore, the backend should be selected with a small disk, a small amount of RAM, but a powerful processor.

In practice, in projects with tens of millions of requests per day, we chose servers for backends of this configuration:

8GB of memory	32 cores	minimum SATA drive	without RAID
On the backends, caching services, such as Memcach, are often placed. This is very convenient, because Similar services are usually demanding on RAM and are not very demanding on the processor. We usually installed 32GB of RAM with the same configuration.

Frontend


Unlike the backend, the front-end server handles connections from a large number of visitors. Connections take up space in the RAM. Therefore, its volume should be large.

In addition, the frontend often performs CPU- intensive tasks - compression of statics and responses from backends, encryption of responses with SSL enabled, authorization, etc. Hence the processor here will also need a powerful one.

The disk most often plays a small role. If all the static is stored in the RAM, the simplest version will suffice. However, if the frontend delivers media content, you should choose faster SAS or SSD disks.

To ensure high availability, front-ends should also be present in at least two pieces.

We usually picked up such servers for front-ends:

32GB of memory	32 cores	minimum SATA drive	without RAID
Database


The database often acts as the most loaded node. The database has high requirements for all resources - both to the processor and to RAM and to the disk.

It is very difficult to predict the form of the load and the amount of data for developing projects. What will be more - a record or a reading? The base will fit into RAM or not? Will there be more samples on the primary key or aggregate? It is unlikely that these questions will be answered in advance. Therefore, the choice is better to do the most common - a more powerful processor, more RAM and a faster disk.

In a good case, about 30% of all data should be stored in RAM - this will ensure good performance when working with the disk in most cases.

RAID must be installed, because The probability of failure of disks here is very high, and the time of downtime should tend to zero.

We usually chose such servers for database maintenance:
64GB of memory	32 cores	2x256GB SSD	RAID 10
Other nodes

On large projects, there is a large number of different nodes, except backend, frontend and databases.

Mail servers

The disk is small, the processor is average, there is not enough RAM. The most modest server. If you send a lot of mail (millions) and use encryption and authorization (DKIM), you should put the server with a powerful processor and a fast disk, increasing the RAM - nothing.

File / media storage

The load on the processor on such servers is low, but the disk and RAM are high. In a simple case, you should rely on the operating cache, put huge disks and more RAM.

Task Queues

If you use something like German that does not save tasks to disk, choose a lot of RAM, a small disk and a simple processor (operations on such a server are very simple). In case of synchronization of tasks with the disk, you should put the disks faster (SAS or SSD), because the number of tasks passing is usually quite large.

Statistic collectors

Usually it is a server that collects data for subsequent analytics (Hadoop, Elastic Search, Vertica, etc.). Very often they are not critical to instantaneous records, but the data is often very very very much. The query form usually involves aggregate samples with a large number of records, so it is almost impossible to effectively use the RAM here. You should choose large drives (SATA for economy, SSD for speed), medium processors and average amount of RAM.

Full-text indexes

Under the task of searching the text often separate technology. Elastic Search, Sphinx or Solr, basically everything looks like a database. Quick disks, more RAM (so that a third of the index is stored in memory) and a powerful processor.

Caching Clusters

There's just a lot of RAM, medium processors (queries in huge quantities will load server processors - hash computing, data compression) and minimal disks.

The most important

Iron is chosen based on three main parameters - processor, memory and disk. The fronts are critical to memory and the processor, the backends to the processor, and the databases to everything at once.

Most often you will make a choice between the processor and RAM. Avoid universal servers - it will be expensive.