Profiling in MySQL

Percona Toolkit is a toolkit that provides advanced tools for managing MySQL, collecting analytical information and processing it, conducting routine operations, restoring data, and so on. Including - tools for profiling MySQL. The package is included in all common Linux distributions of systems:

apt-get install percona-toolkit
After installation, you will have about 20 tools available.

Analysis of slow queries

For profiling MySQL queries, the pt-query-digest tool is used . Before using it, make sure that the slow query log is enabled in my.cnf :

log_slow_queries	/var/log/mysql/mysql-slow.log
long_query_time		1
In order to analyze the log file, you need to perform:

pt-query-audit /var/log/mysql/mysql-query.log> digest; cat digest
After that, a long enough report will be generated. From the top you will see general information about all the slow queries:

# Attribute total min max avg 95% stddev   median
# ============ ======= ======= ======= ======= ======= == ===== =======
# Exec time 634s 1s 54s 4s 8s 6s       2s
# Lock time 7ms 0 953us 48us 167us 151us 0
# Rows sent 107.62M 0 1.52M 720.30k 1.46M 619.39k 462.39k 
# Rows examine 107.62M 0 1.52M 720.30k 1.46M 619.38k 462.39k 
# Query size 70.38k 45 10.94k 471.04 88.31 1.93k    49.17
Pay attention to the medians. In the example, half of all queries are performed longer than 2 seconds - this is bad.

Below you can see the general profile of all slow queries:

# Rank Query ID Response time   Calls R / Call V / M Item
# ==== ================== ============== ===== ====== == === ==============
# 1 0x67A347A2812914DF 440.2687 69.5%    119 3.6997 1.61 SELECT stats_min
# 2 0x0212FEF8C81EF1B6 78.2438 12.3% 8 9.7805 27.12 UPDATE files
# 3 0x2D2411FF42855142 67.4195 10.6% 10 6.7419 33.64 UPDATE projects
# 4 0xE5418AFB28BC18CF 25.1232 4.0% 7 3.5890 2.62 INSERT UPDATE files
All queries are sorted by the total time . Optimize requests in the same order - this will give the greatest effect. In addition, pay attention to the R / Call column - it indicates the average time for execution of one query. In the example for the second query, this time is longer than 9 seconds, you should find out the reasons.

After that, you can see the detailed profile of each request in the report:

# Query 1: 0.00 QPS, 0.00x concurrency, ID 0x67A347A2812914DF at byte 98907
# Scores: V / M = 1.61
# Time range: 2014-12-12 01:00:40 to 2015-02-03 01:01:22
# Attribute pct total min max avg 95% stddev median
# ============ === ======= ======= ======= ======= ====== = ======= =======
# Count 77,119
# Exec time 69 440s 1s 9s 4s 8s 2s 2s
# Lock time       0 0 0 0 0 0 0 0
# Rows sent      99 107.52M 1.19k 1.52M 925.21k 1.46M 565.37k 462.39k
# Rows examine   99 107.52M 1.19k 1.52M 925.21k 1.46M 565.37k 462.39k
# Query size 8 5.68k 46 54 48.89 51.63 1.51 49.17
# String:
# Hosts localhost
# Users root
# Query_time distribution
# 1us
# 10us
# 100us
# 1ms
# 10ms
# 100ms
# 1s ================================================ ================
# 10s +

# Tables
# SHOW TABLE STATUS FROM `t` LIKE 'stats_min' \ G
# SHOW CREATE TABLE `t`.`stats_min` \ G
SELECT * FROM `stats_min` \ G

# Converted for EXPLAIN
# EXPLAIN SELECT * FROM `stats_min` \ G
These data will show the detailed allocation of resources. You should pay attention to Lock time , Rows sent , Rows examine . The more these values, the worse.

The most important

Be sure to enable the slow query log on the production servers. Use the pt-query-audit tool to periodically check for slow requests. Conduct optimization of requests according to the order in which they occur in the report.