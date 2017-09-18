4 main performance teams

How quickly to diagnose the problem on the server? There are several convenient commands for this.

top

The command shows the current tasks that are serviced by the kernel. By default, the top command automatically updates this data every five seconds:

top - 11:17:51 up 153 days, 4:51, 1 user, load average: 0.01, 0.02, 0.05
Tasks: 64 total, 1 running, 63 sleeping, 0 stopped, 0 zombie
% Cpu (s): 0.0 us, 0.0 sy, 0.0 ni, 100.0 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem: 508944 total, 501244 used, 7700 free, 108552 buffers
KiB Swap: 0 total, 0 used, 0 free, 148108 cached

  PID USER PR NI VIRT RES SHR S% CPU% MEM TIME + COMMAND           
    1 root 20 0 10648 708 576 S 0.0 0.1 4: 40.64 init              
    2 root 20 0 0 0 0 S 0.0 0.0 0: 00.02 kthreadd          
    3 root 20 0 0 0 0 S 0.0 0.0 6: 03.05 ksoftirqd / 0       
    5 root 20 0 0 0 0 S 0.0 0.0 0: 00.00 kworker / u: 0       
    6 root rt 0 0 0 0 S 0.0 0.0 0: 00.00 migration / 0       
    7 root rt 0 0 0 0 S 0.0 0.0 1: 28.15 watchdog / 0        
    8 root 0 -20 0 0 0 S 0.0 0.0 0: 00.00 cpuset            
    9 root 0 -20 0 0 0 S 0.0 0.0 0: 00.00 khelper           
   10 root 20 0 0 0 0 S 0.0 0.0 0: 00.00 kdevtmpfs         
   11 root 0 -20 0 0 0 S 0.0 0.0 0: 00.00 netns             
   12 root 20 0 0 0 0 S 0.0 0.0 0: 30.45 sync_supers       
   13 root 20 0 0 0 0 S 0.0 0.0 0: 00.61 bdi-default       
   14 root 0 -20 0 0 0 S 0.0 0.0 0: 00.00 kintegrityd       
   15 root 0 -20 0 0 0 S 0.0 0.0 0: 00.00 kblockd           
   17 root 20 0 0 0 0 S 0.0 0.0 0: 18.19 khungtaskd        
   18 root 20 0 0 0 0 S 0.0 0.0 0: 27.31 kswapd0           
   19 root 25 5 0 0 0 S 0.0 0.0 0: 00.00 ksmd
Nobody uses half of the chips that are available in this team. If you press the h button , the command prompt will open.

With the k button, you can turn off any process by its PID.

Using the x button, you can change the column by which the processes are sorted.

The command displays basic information about processes, processor and memory. The processor time is divided into the following types:

us : time spent for user tasks.
sy : time spent working on the kernel / system.
id : timeout (the processor does nothing).
wa : time spent waiting for disk / network / ...
st : time "stolen" from the virtual machine (the virtualization platform, it's bad when it's big).
vmstat

The vmstat command will show the snapshot of the processor, IO, processes, and memory:

procs --------memory----- swap-- -io- -system-- -cpu-
 rb swpd free buff cache si so bi bo in cs us sy id wa
 0 0 0 7108 108932 148556 0 0 1 14 3 2 1 0 98 0
The first columns show the processes:

r : Processes waiting for the processor
b : processes waiting for the disk / network / user, etc.
.
Both indicators should tend to zero.

The following columns show the use of memory:

swpd : the size of the swap used, it is bad if it is much greater than zero
free : free memory, this should tend to zero
buff : any buffers (for example on I / O operations)
cache : operating system cache
Then comes infa about I / O:

bi : received blocks from the device (disk type)
bo : sent blocks to device
Next are data about interrupts ( in ) and context switches ( cs ), as well as data about the use of the processor (the format is the same as that of the top command).

Vmstat shows a short-term picture, but it's worthwhile to have an idea about long-term trends (this is next).

iostat

First you need to install the command:

apt-get install sysstat
The command shows three reports: CPU usage, i / o utilization and network subsystem. If you run a command without parameters, it will show a minimum of information:

Linux 3.2.0-4-amd64 (ruhighload.com) 10/25/2015 _x86_64_ (1 CPU)

avg-cpu:% user% nice% system% iowait% steal% idle
           0.85 0.00 0.37 0.08 0.71 98.00

Device: tps kB_read / s kB_wrtn / s kB_read kB_wrtn
vda 1.01 0.88 13.92 11683285 184438328
Using devices (device) shows all the connected drives and information about their use. Reading per second (kB_read / s) and writing per second (kB_wrtn / s) will help to give an idea of ​​the load of disks.

free

The command shows the statistics of memory and swap:

             total used        free shared buffers cached
Mem: 508944 478368 30576 0 71780 162704
- / + buffers / cache: 243884 265060
Swap: 0 0 0
 
The amount of used memory (used) in a good case should tend to all available memory (total), however the swap should be minimal (or zero, as with us).

The most important

top, vmstat, iostat and free - 4 commands that will greatly simplify the analysis and tuning of server performance.