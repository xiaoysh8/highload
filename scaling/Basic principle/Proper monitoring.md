Proper monitoring

Any Web application is constantly changing. As from the point of view of the interior (new functions and technologies), and in terms of external conditions (size and activity of the audience). The quality of the application depends critically on correct and timely diagnosis. The dynamics of Web applications translates the diagnostics to a new one - a constant level. It is important not just to know the maximum about the system. It is also important to learn about the changes as quickly as possible. This is a monitoring task.

There are three main components of the monitoring system:

Status monitoring. Check the normal operation of the components.
Monitoring of trends. Collect changes in indicators and their subsequent analytics.
Business monitoring. Observation of deviations in business indicators.
Status monitoring

The task of status monitoring is to constantly check all components of the system for the correctness of their operation. 

For example:

Does the MySQL database work?
Is there free space on the hard drive.
Is Nginx responding correctly to requests?
The most popular solutions:

Monit is the simplest choice. Is able not only to make checks, but also to perform certain actions in emergency situations (for example, it may try to launch a crashed process). Good for small projects.
Nagios - monstrolo. Can do everything. If your system has more than a few dozen components, switch to it.
The main rule of monitoring settings is to check as many system indicators as possible. Here the rule works - the more we know, the better.

Notifications

The main task of status monitoring is to report problems. In practice, this is usually a letter or SMS message in the event of a breakdown. From the correct setting of notifications, the effectiveness of monitoring depends on 90%.

First of all, it is very useful to have a dashboard with the highest possible metrics. Most often this is the availability of the Web server, databases and backends. Those. those nodes that are directly responsible for generating a response to the user's request. 

The setting of notifications usually follows this principle:

Parameter selection . Not all settings need to be configured for notification. Some of them are basic (for example, the availability of a Web server). Some auxiliary (for example, the number of open file descriptors).
Priority setting . Two groups of parameters should be distinguished:
High priority is a critical problem. This should get about 5% of the indicators - those in the deviation of which will be a disaster. Usually this is the availability of all nodes (ping), the CPU utilization rate, free space in RAM and on the disk.
Low priority is a problem that needs to be addressed. These are the remaining metrics that can cause serious problems if they are not reacted in the near future.
Determination of the trigger threshold . For many metrics, two thresholds should be selected - one warning (low priority, for example, 10% of free disk space), and the second one requiring an instant response (high priority, 1% free).
Setting up notifications is not a one-time job. This should be done constantly, because priorities change and new metrics appear. Observe the rules:

Avoid blindness . If you repeatedly received a notification and did not react to it in any way, the notification should be disabled.
Avoid flow . The monitoring system should distract you on important matters, and not send digests about dozens of problems that could not be repaired.
Use different notification mechanisms . For example, SMS is only for the most important cases. Mail messages are for medium importance. The log in the Web interface or a separate mailbox is for low priority notifications.
Include a notice if you are in doubt . It's better to make sure that notification is useless and disable it later.
Consider the variance of the values . Monitoring such indicators as the Load Average can be a problem. This indicator can jump out of limits several times a day simply because of the nature of the loads. Increase the threshold gradually to achieve the usefulness of the notification.
Do a continuous check of the notification system itself . Sometimes the mail can be broken, or the SMS will terminate the deposit. Be sure to configure the delivery of notifications by using a backup monitoring system.
Monitoring trends

Knowledge of the current status of the system is not enough to make predictions. Clearly, the problem is better to prevent than to react to it. This requires systems for collecting and storing historical data on the change in indicators. Such systems work in the same way as status ones, but usually they collect much more indicators and store the entire history of their changes. 

The most popular solutions are:

Munin is a simple tool with the largest repository of ready-made plug-ins.
Cacti - an advanced system for working with graphics including zoom.
Collectd - more productive than its competitors, is designed for a huge amount of data.
Analytics and forecasts

Analytics of historical data will allow to predict the need for scaling. In addition to the usual metrics, such as CPU utilization and the amount of available memory, higher-level indicators should be included here, for example:

The number of requests per second on Nginx, php, MySQL, etc.
The number of threads and processes.
The size of the queues (for example, on the mail server or the system of tasks ).
Time of page generation.
Trend collection systems also allow you to customize thresholds and notifications. Thresholds should be selected slightly lower than in the system of status monitoring. This will allow you to receive advance notice of possible future problems.

Business metrics and Real time tracking

Normal operation of all components of the application does not always mean the proper operation of the application itself. Problems such as non-working registration or incorrect reference in the letter will not be reflected in the mentioned monitoring systems. Many problems can be temporary or limited. For example, the inaccessibility of one of the systems of social authorization or the speed of loading pages for users from a particular region.

That is why there is a need to monitor business metrics. Many analytics systems, such as Google Analytics, allow you to conduct a detailed analysis of historical data. However, such tools are inconvenient to use to detect deviations in real time.

There are tools for simple statistics collection, such as io Track . Integration of such a system occurs by adding a counter to certain events. For example, to track the number of registrations, you need to add code to count their number:

...

# вставка данных нового пользователя 
mysql_query('INSERT INTO users SET email = ...');
$user_id = mysql_insert_id();

# трекинг регистраций
track('signups');
...
# Example of ioTrack interface for [Hd]

Common examples of business-level metrics are:

Activity of the audience (number of visits / views).
The speed of the application (time of loading pages).
Registrations / purchases / comments / other actions.
Conversions of various values ​​(for example, conversion of advertising campaigns in registration).
High-level metrics related to business logic (for example, the number of processed photographs).
Collection of such data will allow to find out deviations not only in the system operation, but also in the environment. For example, spam attacks, which often dramatically change several business metrics, although they may not affect the load of the system.

It is very convenient to display several basic quantities in dashboards on separate monitors in the office. It allows not only to be aware of the problems, but also to receive "live" information about the quality of the application.

The most important

Remember, the monitoring task is to provide information about failures in the work. It is not executed one-time, the changes must be implemented together with the changes of the application itself.

First of all, organize monitoring business metrics. Monitoring trends will predict changes in equipment and avoid catastrophes. Status monitoring is your 911 service, a measure of its effectiveness is the usefulness of notifications.