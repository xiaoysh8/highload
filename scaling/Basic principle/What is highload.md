What is highload

First of all, highload is an extremely relative concept. It is never measured by the number of requests or the speed of the site; there is simply no such thing as an "average site". All sites are specific and the same number of requests can lead to completely different loads on different resources.

What is a high load (read "hayload")? The correct question will be sooner - when the "normal" project ends and highload begins?

When does high load occur?

This moment comes when your current infrastructure begins to show the first signs that it stops coping with the load. If you have a 128 MB VPS - for you it can be 10 requests per second. For someone it can be 10 thousand requests. The bottom line is not in them, but whether there is a need for scaling and optimizing the infrastructure.

If your site does not cope with the load - all, now you are in the club highload.

Symptoms

To understand that there is a problem, it needs to be diagnosed. That's why, at any load, you need a good monitoring system. It will help determine the very moment when it's time to scale.

Good systems for monitoring and tracking server performance trends:

Munin
Zabbix
Nagios
What usually happens when the highload point approaches:

Slow or endless page loading
Random Errors
Disconnected connections from the Web server
Partial content loading (for example, there is no part of the images)
Reducing audience activity
In addition, be sure to use the analytics system in order to understand the impact of your hardware on users:

Google Analytics
Yandex Metrics
What should I do first?

First of all, you need to determine the causes of the problems. Check the following:

Web server

A properly configured Web server will significantly relieve the hardware. If you are using Nginx, make sure it is properly configured . If your site has pictures and other files that never change, make sure that Nginx is properly configured for uploading files .

If you do not use Nginx, I advise you to switch to it. According to statistics, Netcraft nginx served or proxyed 18.16% of the heaviest sites in May 2014.

Optimizing the client part will save a significant amount of resources and ensure a high speed of the resource for users. Even if there are no problems on the server, the site can work much faster.

MySQL

The most common problems are found in databases. Make sure that MySQL is configured for your needs.

Enable the slow query log in MySQL and use the tools to analyze it .

PHP

To the surprise, the performance problems associated with PHP are not so often. However, the analysis of the application (profiling) will identify bottlenecks. For profiling, use the xhProf utility .

Is it worth it to prepare in advance?

If you are working on a site that is gradually growing, then sooner or later you will go into highload mode. Is it worth doing something in advance? Yes.

First of all, make sure that all the simple things that could be done are done. Using the example of basic optimization of VPS, you can get a significant increase in productivity in 10 ... 15 minutes.

From the first days of work include monitoring. Here the rule works - the more you know, the better.

But , in no case should not be engaged in preliminary optimization. Sometimes you can guess. But more often than not, you risk simply losing time optimizing what is likely to change significantly. It is better to focus on the flexibility of the system, which will quickly make the required changes. Know the mistakes to be avoided , and do not make them your own.