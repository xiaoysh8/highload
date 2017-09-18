Running processes in the supervisor

Supervisor is a client / server system through which a user (administrator) can monitor connected processes on UNIX-type systems. The tool creates processes in the form of sub-processes on its own behalf, therefore it has full control over them.Operating scheme supervisor

Supervisor consists of a server part called supervisord , which creates and manages all processes, and the supervisorctl system / web interface for managing and monitoring supervisord .

Install and configure supervisor

The process of installing supervisord in Debian is very simple. It is necessary to execute the command:

apt-get install supervisor
# Install a supervisor, you must have root rights

After installation, the supervisor needs to configure and add the programs / processes that it will manage. The default configuration file is located in /etc/supervisor/supervisord.conf (for Ubuntu, Debian) or /etc/supervisord.conf for other systems (FreeBSD, etc.).

To add a new process (vorker), you need to add the file with the same code:

[program:worker]
command=/usr/bin/php /var/www/worker.php
stdout_logfile=/var/log/worker.log
autostart=true
autorestart=true
user=www-data
stopsignal=KILL
numprocs=1
# Create a vendor to manage the PHP process

Briefly go through the parameters

[program: worker] - the name of the process / vorker, to which all subsequent parameters of the section will apply;
command = / usr / bin / php /var/www/worker.php - command to run the file, that is, the path to the desired file;
stdout_logfile = / var / log / worker.log - output the console to a file;
autostart = true - the start of the vorker with the start of the supervisor;
autorestart = true - restarts the vorker if he fell for some reason;
user = www-data - start the process under a certain user;
stopsignal = KILL is the stop (kill) signal of the process. If not specified, then the default command is used - TERM ;
numprocs = 1 - number of instances of the specified vorker
.
In the case where it is required to start several instances of the same process at once, the configuration will look like:

[program:worker]
command=/usr/bin/php /var/www/worker.php
process_name=%(program_name)s_%(process_num)02d
numprocs=10
stdout_logfile=/var/log/worker.log
autostart=true
autorestart=true
user=www-data
stopsignal=KILL 
# Create 10 process copies

In this case, the line process_name =% (program_name) s _% (process_num) 02d is added , which specifies the names of all copies of the process, in our case worker_00 , worker_01 , etc.

After adding new processes / vorkers, do not forget to restart the supervisor :

/etc/init.d/supervisor restart
# Restart Supervisor

Supervisor also includes a web-based supervisorctl user interface , which is enabled using the configuration file. To do this, you need to change the section [inet_http_server] by typing in the correct username and password:

[inet_http_server]
port=127.0.0.1:9001
;username=some_user_name
;password=some_password
# Enable the supervisorctl Web Console on the 9001th socket

Now all available processes can be controlled through the browser. Remember that after the configuration change, the supervisor and / or supervisorctl need to be updated.

Additional Features

The supervisor has a built-in event monitoring mechanism, through which the system can notify you of errors:

[eventlistener:memmon]
command=memmon -a 200MB -m error@ruhighload.com
events=TICK_60
# If the process consumes more than 200 MB of memory, memmon restarts it and sends a notification to the mail, checking every 60 seconds

With the help of events and your own script on Pyton, you can check out virtually every aspect of the desired process.

The most important thing

Supervisor is a simple and powerful tool for process control. With the correct configuration, it is able to ensure the smooth operation of your web service.