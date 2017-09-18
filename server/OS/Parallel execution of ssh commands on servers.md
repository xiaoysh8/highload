Parallel execution of ssh commands on servers

When managing a large number of servers, you often have to run the same command on several servers (dozens / hundreds / thousands).

There are many tools for solving this problem. In .io we use a simple pssh solution .

On Ubuntu, this tool is available in packages:

apt-get install pssh
Before using the tool, you need to prepare text files with server addresses in the format:

10.10.0.1
10.10.0.2
10.10.0.3
...
# List of IP servers for executing commands using pssh

We actively use the micro-service architecture, so we have dozens of services. Therefore, we created a common file with a list of all servers and a file for each service:

all.txt stores a list of all the servers on the system. It is used extremely rarely.
serviceX.txt stores a list of all servers of a particular service.
To execute the command on all servers:

parallel-ssh -h all.txt -o /tmp/ssh ls -la
# Execute the command "ls -la" on all servers from the file all.txt

The result of the command will be written to the / tmp / ssh folder - into a separate file for each server:

# ls -la / tmp / ssh /
-rw-r - r-- 1 root root 1238 Aug 22 16:46 10.10.0.1
-rw-r - r-- 1 root root 1269 Aug 22 16:46 10.10.0.2
-rw-r - r-- 1 root root 1110 Aug 22 16:46 10.10.0.3
-rw-r - r-- 1 root root 950 Aug 22 16:46 10.10.0.4
To use authorization on keys:

parallel-ssh -h all.txt -x "-i key.rsa" -o /tmp/ssh ls -la
# key.rsa will use to access all servers

You can manually set the number of threads to execute commands:

parallel-ssh -h all.txt -p 10 -o /tmp/ssh ls -la
# commands will be executed in 10 threads

TL; DR

Use pssh to execute commands on a large number of servers:

parallel-ssh -h all.txt -o /tmp/ssh ls -la
# In all.txt you need to add a list of servers, each ip address on a new line