Testing the load on the site

When a web application (site, service) is still young enough, but aimed at a broad audience, it is very difficult to understand how powerful server equipment is required. So the best solution is to simulate the flow of users using synthetic tests.

Apache Bench

Probably one of the easiest to use and popular tests to check the load on the site. The utility is suitable for both simple and advanced testing:

ab -c 50 -n 10000 -f TLS1.2 -H "Accept-Encoding: gzip,deflate" https://somesite.com/
# Checking the maximum number of queries with TLS

The team completed 10,000 requests in 50 threads and, among other things, showed the speed and the processed number of requests:

Total transferred:      59560000 bytes
HTML transferred:       52160000 bytes
Requests per second:    816.77 [#/sec] (mean)
Time per request:       122.434 [ms] (mean)
Time per request:       2.449 [ms] (mean, across all concurrent requests)
Transfer rate:          2375.33 [Kbytes/sec] received
# The test log gives much more information

From this report, the most important data will be:

Requests per second - the number of requests per second. For example, if a page consists of 20 parts (CSS, images and HTML), in our example the server is capable of processing about 40 simultaneous users per second.
Time per request (mean) - the average time for execution of a group of parallel requests (in our case 50);
Time per request (mean, across all concurrent requests) - the average time for execution of one query.
AB is useful for a quick and rough assessment of the performance of the web server, so if you need to get more close to reality data, you will have to use additional utilities.

Httperf

This open-source test was developed by HP to measure the performance of a web server. The tool has not been updated for several years, but it is still very relevant.

The utility, like ab, is easy to use and has quite a lot of functionality. It runs as simply as ab:

httperf --hog --server 192.168.122.10 --wsess=100000,5,2 --rate 1000 --timeout 5
# Create 100,000 sessions (5 calls every 2 seconds) at a speed of 1000

And the log will look like this:

Total: connections 117557 requests 219121 replies 116697 test-duration 111.423 s

Connection rate: 1055.0 conn/s (0.9 ms/conn, <=1022 concurrent connections)
Connection time [ms]: min 0.3 avg 865.9 max 7912.5 median 459.5 stddev 993.1
Connection time [ms]: connect 31.1
Connection length [replies/conn]: 1.000

Request rate: 1966.6 req/s (0.5 ms/req)
Request size [B]: 91.0

Reply rate [replies/s]: min 59.4 avg 1060.3 max 1639.7 stddev 475.2 (22 samples)
Reply time [ms]: response 56.3 transfer 0.0
Reply size [B]: header 267.0 content 18.0 footer 0.0 (total 285.0)
Reply status: 1xx=0 2xx=116697 3xx=0 4xx=0 5xx=0

CPU time [s]: user 9.68 system 101.72 (user 8.7% system 91.3% total 100.0%)
Net I/O: 467.5 KB/s (3.8*10^6 bps)
# Among other things, the performance shows the rate of request (Request rate)

This report should focus on:

Connection rate - the real speed of creating new connections. It shows the server's ability to handle connections, that is, in our case up to 1055 s / s, but not more than 1022 simultaneous connections.
Connection time [ms] - the lifetime of successful connections between initialization and closing. Again, it shows the performance of the server when processing a large number of connections.
Request rate - the speed of processing requests. That is, the number of requests that the server can perform in a second, shows the responsiveness of the web application.
But for a deeper check and a significant load will have to use even more advanced tools.

Tsung

This is a powerful, advanced, multitasking and multi-threaded utility. The tool can be used to load HTTP servers, WebDAV, SOAP, PostgreSQL, MySQL, LDAP and Jabber / XMPP. Supports SSL, monitoring of system resources and SNMP, Munin or Erlang agents on remote servers, simulation of users behavior and extended reports.

The tool is written in Erlang, so first you need to install the necessary repositories, and then download and install Tsung:

wget http://tsung.erlang-projects.org/dist/tsung-1.6.0.tar.gz
tar zxfv  tsung-1.6.0.tar.gz
cd tsung-1.6.0
./configure && make && make install
# Unpacking and compiling the utility

All the settings of the tool must be written in its configuration file:

cp  /usr/share/doc/tsung/examples/http_simple.xml /root/.tsung/tsung.xml
# Copy the configuration file template to the Tsung directory

After that, you need to edit it by specifying the necessary parameters:

<?xml version="1.0"?><tsung loglevel="notice" version="1.0">

  <clients>
    <client host="localhost" use_controller_vm="true" maxusers="10000"/>
  </clients>

<servers>
	<server host="192.168.122.10" port="80" type="tcp"/>
</servers>

<load>
	<arrivalphase phase="1" duration="1" unit="minute">
		<users maxnumber="10000" interarrival="0.05" unit="second"/>
	</arrivalphase>
</load>

 <sessions>
<session name="stack" probability="100" type="ts_http">
   <for from="1" to="10000" var="i">
    <request><http url="/stack/" version="1.1" method="GET"/></request>
   </for>
  </session>
 </sessions>
</tsung>
# You can specify additional options (for example, users' browsers), multiple nodes for user simulation

Now you can run tsung:

tsung -f tsung.xml start
Starting Tsung
"Log directory is: /root/.tsung/log/20160428-1117"
# To run from multiple nodes, you must first specify them in the settings

When the utility has finished its work, you can view the report:

mkdir report_output
cd report_output
/usr/lib/tsung/bin/tsung_stats.pl --stats /root/.tsung/log/20160428-1117/tsung.log
chromium graph.html 
# Specify the preferred browser

The report will consist of graphs and important additional information. It is worth paying attention to:

Session - the total number of users and the number of simultaneous sessions per second that the web server has processed.
Request - the response time of the web server, its ability and speed of processing simultaneous requests. For example, 200 requests / s means that on average 10 users can simultaneously get to a web page consisting of a total of 20 components (CSS, pictures and HTML). And this is more than 400 000 visitors in 12 hours.
Connect - the time required for the connection, that is, the responsiveness of the web server .
Additional graphs will allow you to evaluate the load on the web server for the entire test time, track errors and dynamics.

Other utilities

Of course, the list of tools for checking the performance of the web server and testing the load on the site is not limited to the ones given in this article. There are a lot of such utilities, both paid and free. There are sites for load generation, such as LoadImpact, utilities to run from the command line and full programs with GUI. One of the most popular with the user interface, by the way, is the Apache JMeter - powerful, advanced and quite complex.

The most important thing

Apache Bench, Httperf and Tsung are great for testing the load on large and small sites. But only tsung will be able to squeeze all the juice from the web server and show what it is capable of in conditions close to reality. Do not forget that all the tests must first be done for one user in order to trace the dependence and have a reference point.