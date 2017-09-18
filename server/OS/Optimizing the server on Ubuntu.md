Optimizing the server on Ubuntu

Tuning the system parameters is used to improve server responsiveness and performance. Do not rush to buy new hardware, maybe it's all about software. First, analyze the load on the server and test your website for workload . file system

File system

First, make sure you are using the ext4 file system :
df -T


# Вывод будет похожим на:
Filesystem     Type     1K-blocks    Used Available Use% Mounted on
/dev/vda1      ext4      20511592 2494920  16960708  13% /
tmpfs          tmpfs       250152       0    250152   0% /dev/shm
tmpfs          tmpfs         5120       0      5120   0% /run/lock
# Will be displayed in the Type column

If the file system is Ext3, then it is recommended to translate it to Ext4 - a more productive and improved version.

Virtual memory

To perform time-consuming tasks using virtual memory ( swap ) - when the RAM is filled, part of the program is transferred to the hard disk. Thus, you can use more RAM than there is in the system.Ubuntu swap

The approach does not make sense in systems with a large amount of RAM, especially since the RAM is faster than a constant one. Swap is also not recommended for use on a server with SSD disks - it significantly reduces the lifespan of the repository (frequent write and read processes).

Ubuntu defaults to uploading data when the RAM is full by 40%.

To configure swap, use the parameter vm.swappiness , the value of which must be entered or edited in the configuration file sysctl.conf :

nano /etc/sysctl.conf


# Добавить строчку
vm.swappiness=10
# The paging file is only enabled if free 10% of RAM

After that, you need to save the file and reboot the system. To verify that the new parameter is involved, you must:

sysctl -p


# Или

sysctl -a | grep vm.swappiness
# In the first case, it applies all the parameters of the file, in the second case it selects the desired parameter among all possible variables

Caching improves system performance. But if the web server produces a large number of read / write operations, additional caching can slow down I / O.

Caching parameters can be configured:

$ sysctl -a | grep dirty
 vm.dirty_background_ratio = 10
 vm.dirty_background_bytes = 0
 vm.dirty_ratio = 20
 vm.dirty_bytes = 0
 vm.dirty_writeback_centisecs = 500
 vm.dirty_expire_centisecs = 3000
# Parameters that are responsible for dirty pages - data that needs to be written to disk or sent to swap

Parameters mean:

vm.dirty_background_ratio is the percentage of system memory that can be filled in dirty pages before background processes pdflush / flush / kdmflush write them to disk;
vm.dirty_ratio - the maximum amount of system memory that can be filled in dirty pages;
vm.dirty_background_bytes and vm.dirty_bytes - the two previous items, only in bytes; parameters are interchangeable;
vm.dirty_expire_centisecs - the time that data can be stored in the cache, in our case 30 seconds;
vm.dirty_writeback_centisecs - how often processes pdflush / flush / kdmflush check the cache.
The amount of data that is waiting for an entry can be viewed like this:

cat /proc/vmstat | egrep "dirty|writeback"

 nr_dirty 878
 nr_writeback 0
 nr_writeback_temp 0
# 878 "dirty" pages waiting for a record

To reduce the size of the cache in order to reduce the probability of losing important data in the event of a failure and to minimize possible write / read delays, you need to edit the parameters vm.dirty_background_ratio and vm.dirty_ratio:

vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
# Values ​​are written to sysctl.conf

IPv6

A mixed IPv4 / IPv6 environment can malfunction the networked programs due to unintentional protocol interaction. For example, if you fail to connect apt or ssh to an IPv6 network, incompatible devices.

To disable IPv6, you must:

sudo sh -c 'echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6'
# Temporarily disable IPv6 on all interfaces

And to permanently disable the protocol, you need to edit the /etc/sysctl.conf file:

# Отключение на всех интерфейсах
net.ipv6.conf.all.disable_ipv6 = 1


# Отключение на определенном интерфейсе
net.ipv6.conf.eth0.disable_ipv6 = 1
# To apply new parameters, just type sudo sysctl -p /etc/sysctl.conf

Processes

ubuntu processes
Background processes can seriously "clog up" the system memory. The sysv-rc-conf utility comes to the rescue :

sudo aptitude install sysv-rc-conf
# Installing the tool

sysv-rc-conf allows you to disable unnecessary services to speed up and optimize system resources. 
sysv-rc-conf interface















After a clean installation, Ubuntu Server contains a minimum of services, only the most necessary. But if the server is used for a long time, the list of processes can be extensive and will depend on the programs that you installed.

If the desktop version is used, here is a small list of services that can be disabled (if the system works as a server):

alsa and alsa-utils - sound subsystems;
atd - the scheduler, it is not necessary, if there is a cron;
bluez-utiles - Bluetooth service;
cupsys - printer management subsystem;
dns-clean - DNS clearing service when using dial-up;
fetchmail - e-mail delivery service;
gdm - desktop manager (GUI);
gdomap - GNUstep support service;
hibernate - the service of hibernation;
hotkey-setup - support for hot keys;
hotplug and hotplug-net - hot-plug devices;
ifrename is the naming service for network interfaces;
laptop-mode - laptop mode, not needed on the server;
ppp and ppp-dns - services for connecting via modem;
winbind, smbd and nmbd - part of Samba, is needed for general access with devices under Windows.
The main recommendation - do not disable unknown processes, use the manual and Google.

The most important thing

Before starting optimization, you need to diagnose the system and identify weaknesses. Thinking unthinking can only exacerbate the situation. Profiling is helpful in identifying problem areas when running a web server . Optimize web server settings ( Nginx and Apache ), implement HTTP / 2 .