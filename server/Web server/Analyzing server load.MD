## 负载分析可以快速定位性能瓶颈，并作为系统硬件升级的参考依据 

任何服务器硬件包含以下主要部分：

- CPU
- 内存RAM
- 硬盘Hard drives
- 网卡Network interface
- 服务组件Server components

通过搜集统计上面各部分的数据，得到负载分析结果。

CPU

首先应该先检查服务器cpu的运行情能，最快的方法是用'top'命令：

top - 22:20:45 up 67 days,  8:04,  2 users,  load average: 0.05, 0.03, 0.05
Tasks:  88 total,   1 running,  87 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:    508936 total,   461096 used,    47840 free,    43056 buffers
KiB Swap:        0 total,        0 used,        0 free,   237892 cached

  PID USER      PR  NI  VIRT  RES  SHR S  %CPU %MEM    TIME+  COMMAND           
 8867 www-data  20   0 80316 3256 1012 S   0.3  0.6   0:16.06 nginx
30809 mysql     20   0  844m  63m 3332 S   0.3 12.9  38:30.93 mysqld   

注意高亮指示，除了特别用于图形或视频压缩的服务器，cpu负载正常情况下不应该超过10-20%，以下是负载分析数据当中的重点：

- us 表示用户进程。数值高意味着应用进程占用很大服务器资源。
- id 表示cpu空闲时间，这个数值正常应该在80-100之间。
- wa 表示I/O读写操作等待时间。如果指标高，则表明cpu需要花费大量时间等待读写设备相应，大多情况下，这是因为存在大量硬盘操作占用引起的。

mpstat

安装'sysstat'包，使用命令'mpstat', 可以获得更详细的统计数据：

apt-get install sysstat
mpstat -P ALL
It shows data for all CPUs on your server:

10:30:26 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10:30:26 PM  all    0.47    0.00    0.22    0.07    0.00    0.01    0.03    0.00    0.00   99.20
10:30:26 PM    0    0.47    0.00    0.22    0.07    0.00    0.01    0.03    0.00    0.00   99.20
htop

'htop'命令显示的方式更美观：

apt-get install htop
htop
  CPU[||                                               11.3%]     Tasks: 62, 93 thr; 1 running
  Mem[|||||||||170/497MB]     Load average: 0.15 0.05 0.06 
  Swp[                                                       0/0MB]     Uptime: 67 days, 08:22:09

  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
 9377 root       20   0 24596  2064  1372 R  0.5  0.4  0:00.22 htop
 9458 www-data   20   0  315M  7328  2824 S  0.0  1.4  0:00.50 php-fpm: pool www

###分析CPU负载

如果CPU负载超过20%，应该考虑优化程序，如果已经优化过了，那就必须增加新的服务器。

如果系统I/O等待时间很长，那就应该继续分析检查硬盘和网络子系统。


###内存RAM

首先，确定已用和可用的内存数：


free命令, 显求内存占用数据：

	             total       used       free     shared    buffers     cached
	Mem:        508936     480540      28396          0      47552     258912
	-/+ buffers/cache:     174076     334860
	Swap:            0          0          0
\# 'free'命令的结果

重要的数据是free和swap,前者指的是空闲内存，后者指的是内存用光后，系统使用硬盘作为缓存。


查看更详尽的内存使用情况可以用以下方法：

cat /proc/meminfo

	MemTotal:         508936 kB
	MemFree:           28148 kB
	Buffers:           47836 kB
	Cached:           259388 kB
	SwapCached:            0 kB
	Active:           245556 kB
	Inactive:         178600 kB
	...
	SwapTotal:             0 kB
	...
####内存使用分析

以上的情况，内存小不会有问题，不过应该运行相应的服务器监控程序，随着缓存使用的增加，应该考虑增加内存，增加新服务器，分担负载。 缓存使用量，应该保持0 。

###硬盘Hard drives


应用程序访问文件，就会给硬盘带来一定的负荷，另外，数据库相关的操作，会引起持续的硬盘读或写操作。
Start with checking free disk space:

来检查一下剩下的磁盘空间有多少：
df -h
显示所有的分区信息：

Filesystem                 Size  Used Avail Use% Mounted on
rootfs                      20G  2.4G   17G  13% /
udev                        10M     0   10M   0% /dev
tmpfs                      5.0M     0  5.0M   0% /run/lock
Use column shows occupied space. Be sure that this value doesn't exceed 90% for main partitions.

iotop

'iotop' tool is used to display disk load details.
'iotop'命令显示硬盘负载情况，同时也显示不同进程对硬盘的占用情况。
apt-get install iotop
iotop

Total DISK READ :       0.00 B/s | Total DISK WRITE :       2.10 M/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:     218.22 K/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                       
30835 be/4 mysql       0.00 B/s    2.95 K/s  0.00 %  0.07 % mysqld
10273 be/4 www-data    0.00 B/s  194.63 K/s  0.00 %  0.00 % php-fpm: pool www
In this example, two processes — mysqld and php-fpm (php application) are performing write operations to the disk. Close attention should be paid to:
上面的例子里，mysqld和php-fpm两个进程对硬盘进行了写操作。应该重点留意下面的参数：
Actual DISK READ实际硬盘读取 — 单位时间从硬盘读取的全部数据量，和Total DISK READ不同，后者包含硬盘缓存和低层操作优化。 
Actual DISK WRITE实际硬盘写入 — 单位时间写入硬盘的总数据量，与Total DISK WRITE 不同，原因同上。

###分析

####读操作

If HDD gets big amounts of read operations, best ways to act are:
如果硬盘有大量的读操作，最佳处理方式如下：
- 如果主要的读操作是应用本身，那最好启用APC缓存(php)，如果应用读取大量文件，考虑把这些数据放入缓存。如果是由数据库产生的数据，要确保数据库有优化配置。
- 如果数据读取来自web server访问，确认使用HTTP缓存。

####写操作

大量的读写操作，意味着项目的软件或硬件需要升级。Lots of disk write operations usually indicate the need to scale your project.

确保关掉所有的访问和调试日志。如Nginx的access_log。如果大多数硬盘操作，来自数据库，那要考虑把它们放在一个单独的服务器上。另外，可以用主从方式进行扩展。

文件下载也有大量的写操作，诸如此类的扩展细节，可以查看图片存储的例子。


网络

服务器带宽
'cbm' 命令可以监视实时的网络流量：

apt-get install cbm
cbm

它量示每秒所有发出和接收到的数据量：

Interface                            Receive                           Transmit                          Total
lo                                   10.07 MB/s                        10.07 MB/s                        20.13 MB/s
eth0                                 73.15 kB/s                         2.44 MB/s                         2.51 MB/s

流量比较高，不是问题，但如果接近最大值，应该考虑很快进行扩展。如：平均95Mbps，100Mbps带宽，意味着当前的带宽很快就不够用了。


一般分析

'dstat'命令，给出实时的总的服务器信息：

apt-get install dstat
dstat

它按每秒一次，输出系统实时信息

----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
  0   0  99   0   0   0| 873B   18k|   0     0 |   0     0 |  33    75 
  0   0 100   0   0   0|   0    48k| 640B 1550B|   0     0 |  31    80 
  0   0 100   0   0   0|   0     0 | 422B 1110B|   0     0 |  29    66 
  0   0 100   0   0   0|   0     0 | 590B 1110B|   0     0 |  32    70 
  1   0  99   0   0   0|   0     0 | 478B 1110B|   0     0 |  31    66 
  0   0 100   0   0   0|   0     0 | 814B 1110B|   0     0 |  34    63 
  0   0 100   0   0   0|   0     0 | 814B 1110B|   0     0 |  39    70 
  0   0 100   0   0   0|   0    20k| 814B 1110B|   0     0 |  40    75 

请注意：

total-cpu-usage — cpu负载
dsk/total — 硬盘负载
net/total — 网络负载

最重要的是，记住防范问题发生，所以，要自动监控系统的相关数据，提供实时预警。