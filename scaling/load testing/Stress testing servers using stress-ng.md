Stress testing servers using stress-ng

Stress testing is convenient for both cloud and physical servers. If you work in the clouds, you can understand the real bandwidth of the node. If you deploy the system on a physical hardware - you can make sure that everything is fine with the iron.

In .io, we use more than 100 physical servers (except cloud servers) to maintain the storage and processing of statistics. We sometimes encounter bugs even on new servers. Problems can be different - the cooler on the processor, because of which the processor during operation can go into the mode of throttling (artificial lowering of the frequency). Disk with bugs (one of two in RAID'e). Or else some kind of dregs.

Stress test allows you to generate a load on all key subsystems of the server. This makes it possible to verify that all components are functioning normally.

Tools for stress tests

There are a whole bunch of tools for stress testing, we settled on stress-ng (we use Ubuntu as an OS).

Note that the stress test is an internal test. Unlike load testing tools, for example ab , stress tests do not consist in generating requests for external services.

The tool is installed from packages:

apt-get install stress-ng
To run a stress test, you must select the type of test and specify its parameters.

Testing the CPU

First of all, we want to test the processor.

stress-ng --class cpu --sequential 8 --timeout 60s --metrics-brief
# we conduct a processor test in 8 threads

In this test, stress-ng will perform different tests from the cpu group in 8 threads (since we have 8 cores), each with a duration of 60 seconds. For testing, various methods will be used consistently (for example, counting the total greatest divisor or Ackerman function). Not only mathematical operations, but also sorting, encryption, compression, searching, working with strings, etc. will be performed.

During the tests, all the cores will be maximally loaded (the result of htop ): 

After the test, we will see the statistics of performance of all 22x processor tests:

stress-ng: info: [6073] stressor bogo ops real time usr time sys time    bogo ops / s bogo ops / s
stress-ng: info: [6073] (secs) (secs) (secs) (real time) (usr + sys time)
stress-ng: info: [6073] af-alg 115214848 60.03 5.24 474.46 1919440.07 240181.05
stress-ng: info: [6073] bsearch 156484 60.00 479.60 0.00 2608.01 326.28
stress-ng: info: [6073] context 2548147 60.00 240.41 239.15 42469.14 5313.51
stress-ng: info: [6073] cpu 115337 60.02 479.75 0.00 1921.49 240.41
stress-ng: info: [6073] cpu-online 458 60.59 0.55 3.15 7.56 123.78
stress-ng: info: [6073] crypt 51219 60.00 479.58 0.00 853.61 106.80
stress-ng: info: [6073] getrandom 1255727 60.00 0.01 478.74 20928.74 2622.93
stress-ng: info: [6073] heapsort 1789 60.00 479.55 0.01 29.82 3.73
stress-ng: info: [6073] hsearch 1486425 60.00 479.49 0.00 24773.70 3100.01
stress-ng: info: [6073] longjmp 43395789 60.00 479.61 0.00 723263.64 90481.41
stress-ng: info: [6073] lsearch 3431 60.00 479.61 0.00 57.18 7.15
stress-ng: info: [6073] matrix 1403584 60.00 479.57 0.00 23393.08 2926.76
stress-ng: info: [6073] mergesort 62685 60.00 478.97 0.02 1044.75 130.87
stress-ng: info: [6073] qsort 4505 60.00 479.50 0.01 75.08 9.40
stress-ng: info: [6073] rdrand 93458694 60.00 479.64 0.00 1557645.83 194851.75
stress-ng: info: [6073] str 16492215 60.00 479.59 0.00 274870.39 34388.15
stress-ng: info: [6073] stream 26741 60.01 477.74 0.05 445.62 55.97
stress-ng: info: [6073] tsc 558813483 60.00 479.65 0.00 9313568.26 1165044.27
stress-ng: info: [6073] tsearch 3540 60.04 479.61 0.00 58.96 7.38
stress-ng: info: [6073] vecmath 10179903 60.00 479.60 0.00 169665.15 21225.82
stress-ng: info: [6073] wcs 4452607 60.00 479.58 0.00 74210.12 9284.39
stress-ng: info: [6073] zlib 7667 60.16 480.79 0.02 127.43 15.95
# metrics after running the test

Statistics will include the name of the test and the figures for the speed of operations. Absolute values ​​have no special significance. However, they should be compared with the numbers of servers of a similar configuration. Especially the numbers in the bogo ops / s columns .

Testing the RAM

For RAM, there is a group of tests, which include the operations of selecting, copying and clearing memory. In addition, this set includes some tests from the class cpu . For example, a compression and sorting test.

stress-ng --class memory --sequential 8 --timeout 60s --metrics-brief
# test group for RAM

During this test we will observe another picture - a large amount of RAM (and swap) will be occupied: 

After graduation, we'll see a summary of the tests passed:

stress-ng: info: [22818] stressor bogo ops real time usr time sys time    bogo ops / s bogo ops / s
stress-ng: info: [22818] (secs) (secs) (secs) (real time) (usr + sys time)
stress-ng: info: [22818] brk 11133716 60.57 2.17 76.51 183827.09 141506.30
stress-ng: info: [22818] bsearch 153344 60.00 477.83 0.02 2555.66 320.90
stress-ng: info: [22818] hsearch 1481213 60.00 477.78 0.01 24686.84 3100.13
stress-ng: info: [22818] lsearch 3252 60.01 478.05 0.01 54.19 6.80
stress-ng: info: [22818] malloc 29479834 60.09 450.03 25.40 490593.76 62006.68
stress-ng: info: [22818] memcpy 113027 60.00 475.77 0.01 1883.73 237.56
stress-ng: info: [22818] mincore 17446227 60.00 42.99 434.86 290770.63 36509.84
stress-ng: info: [22818] null 3230050466 60.00 97.80 380.23 53834181.55 6757003.67
stress-ng: info: [22818] pipe 189110658 60.00 95.79 382.06 3151840.67 395753.18
stress-ng: info: [22818] qsort 4283 60.03 478.03 0.00 71.35 8.96
stress-ng: info: [22818] tsearch 4140 60.03 477.78 0.00 68.96 8.67
stress-ng: info: [22818] vm 8546560 60.00 473.77 3.71 142436.13 17899.30
stress-ng: info: [22818] vm-rw 21238 60.01 0.21 469.21 353.92 45.24
stress-ng: info: [22818] zero 1118192157 60.00 38.27 439.75 18636544.45 2339216.26
# metrics after running the test

Testing disks

There are two groups of tests that are worth accomplishing. First - a group of testing low-level I / O devices:

stress-ng --class io --sequential 8 --timeout 60s --metrics-brief
This group of tests includes creating / deleting files, writing blocks to files and synchronizing data in files with a disk. Test result:

stress-ng: info: [11309] aio 128372945 60.00 69.77 408.84 2139548.19 268220.36
stress-ng: info: [11309] aio-linux 1927396 60.13 258.22 220.36 32055.94 4027.32
stress-ng: info: [11309] hdd 786432 74.18 0.21 28.70 10601.49 27202.77
stress-ng: info: [11309] readahead 441786979 60.15 38.26 404.91 7344380.46 996879.25
stress-ng: info: [11309] seek 1568009 60.02 1.04 82.87 26124.47 18686.80
stress-ng: info: [11309] sync-file 8876 60.01 21.13 117.76 147.92 63.91
In addition, it makes sense to run a stress test file system. It includes creating / deleting files and folders, navigating through the file tree, creating links, locking, renaming, etc.

stress-ng --class filesystem --sequential 8 --timeout 60s --metrics-brief
During the testing of the disk subsystem, you can observe the load on the disk with the iostat utility :

iostat 5-y
# Displays the statistics of the drives for the last 5 seconds


The result of running the file system tests:

stress-ng: info: [12865] stressor bogo ops real time usr time sys time    bogo ops / s bogo ops / s
stress-ng: info: [12865] (secs) (secs) (secs) (real time) (usr + sys time)
stress-ng: info: [12865] chdir 19830 60.10 5.75 460.77 329.96 42.51
stress-ng: info: [12865] chmod 806679 60.00 1.38 372.56 13444.65 2157.24
stress-ng: info: [12865] dentry 4917506 60.04 19.03 194.20 81897.94 23061.98
stress-ng: info: [12865] dir 1228800 60.59 3.05 54.29 20280.12 21430.07
stress-ng: info: [12865] dup 2063319701 60.00 110.33 369.29 34388683.03 4301988.45
stress-ng: info: [12865] eventfd 144698784 60.00 34.29 445.27 2411643.14 301732.39
stress-ng: info: [12865] fallocate 1600 60.01 0.35 13.36 26.66 116.70
stress-ng: info: [12865] fcntl 344187771 60.00 132.90 346.80 5736472.46 717506.30
stress-ng: info: [12865] fiemap 153321 60.02 1.02 414.63 2554.48 368.87
stress-ng: info: [12865] flock 9337301 60.00 0.97 313.68 155621.73 29675.20
stress-ng: info: [12865] fstat 272440 60.03 0.40 4.99 4538.28 50545.45
stress-ng: info: [12865] getdent 22444354 60.00 13.87 463.80 374073.37 46987.15
stress-ng: info: [12865] iosync 1106646 60.00 0.66 136.86 18444.01 8047.16
stress-ng: info: [12865] inotify 6089 60.01 15.80 0.00 101.47 385.38
stress-ng: info: [12865] lease 38122684 60.00 26.03 453.26 635375.77 79539.91
stress-ng: info: [12865] link 35471509 60.04 41.14 217.34 590749.75 137231.16
stress-ng: info: [12865] lockf 12893276 60.00 2.33 477.04 214886.95 26896.29
stress-ng: info: [12865] mknod 9016351 60.07 22.98 218.11 150108.30 37398.28
stress-ng: info: [12865] open 170092328 60.00 7.15 471.97 2834874.01 355009.87
stress-ng: info: [12865] procfs 8 60.01 10.82 468.47 0.13 0.02
stress-ng: info: [12865] rename 8 0.00 0.00 0.00 43039.19 0.00
stress-ng: info: [12865] symlink 1698219 60.34 5.84 133.24 28145.33 12210.38
stress-ng: info: [12865] sync-file 15609 60.01 33.81 188.49 260.13 70.22
stress-ng: info: [12865] utime 113807443 60.00 8.75 470.62 1896793.03 237410.44
stress-ng: info: [12865] xattr 45397 60.01 12.90 181.27 756.55 233.80
TL; DR

Use stress testing tools to test the performance of the iron and understand its real performance. stress-ng is a convenient tool for testing servers on Ubuntu. If you just need to run all the tests, use the command:

stress-ng --sequential 4 --timeout 60s --metrics-brief
# Run more than 40 different server stress tests

During the tests, it is best not to touch the computer. After the statistics are executed, it will be displayed directly in the console.