Distributed monitoring and diagnostics / Netdata

Today, there are many paid and free server monitoring systems. We are used to the fact that monitoring is centralized. There is a server with graphs. There are servers that send their metrics to the primary server.

This approach is not very well suited when the task of diagnostics in real time is. Delay in the arrival of metrics can be minutes. Therefore, we open the console to view the current server metrics.

In addition, the monitoring server itself has limited bandwidth. And with the growth in the number of servers served, it is slower.

Netdata is a simple utility for monitoring the server. It is equipped with a large number of plug-ins (Mysql, Nginx and a bunch of others) and an excellent visualization system:netdata

Local data collection

The peculiarity of the Netdata system is that data collection and visualization occur locally on the server. This provides two advantages:

The delay in visualizing the data is a maximum of a second. So, you see the real current picture on the server.
It is easy to scale to hundreds and thousands of servers, because it grows linearly along with their number.
Installation

To install on Linux, just run this command:

bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)
# Installation will take 10 ... 15 seconds

After that, the interface will be available at http://127.0.0.1:19999/ . See the example on XD - the open statistics of our server.

What if there are more than one server?

This system does not provide for viewing aggregated metrics (such as "the sum of all requests to Mysql") from all servers. After opening the interface on the new server, it will go into your personal browser history in the menu "my-netdata".

What if the server is broken?

The use of external monitoring systems can not be avoided. Checking the serviceability of the server should be carried out outside the server itself. But with such a system, the requirements for external monitoring are greatly simplified.

Additional Features

Their whole heap. There is a system of notifications about problems that can be customized. It is possible to send your own metrics (it works like a statsd server). There are a lot of plug-ins and great documentation .

Convenient mode of updating the graphs: 

TL; DR

Try Netdata for real-time server monitoring. To install on Linux:

bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)