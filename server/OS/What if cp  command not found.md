What if cp: command not found

Sometimes a simple problem arises when creating shells on a shell - the scripts work if they are run manually, but they do not work if they are run on the crone or in some other environment. In this case we see the following error:

./task.sh: line 10: cp: command not found
This is due to the fact that in this environment there are other settings for the variable $ PATH . This variable specifies the search path for detecting and executing commands without an absolute path.

How to confirm the problem

Let's take a look at the problem with an example of a script that crashes on a cp command . Let's run a simple command to check the value of this variable:

$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
And now we'll find out where the cp command is located:

whereis cp
cp: /bin/cp
If we start the output of the $ PATH variable on the crown, we can confirm that there is no folder in it with the cp command .

How to solve a problem

To make the script work on the krone, there are several ways. The easiest way is to add the absolute path to our commands to our commands:

cp file /dir # вот это фейлится
/bin/cp file /dir # вот это работает
Or redefine the variable at the beginning of the script:

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
Or do it in Crontab configuration before calling the script:

* * * * * PATH=$PATH:/usr/local/bin /path/to/script.sh
TL; DR

If the scripts that were started manually do not run on the crone or in another environment, it's likely a problem in the $ PATH variable . To solve this problem, it is necessary either to override the value of the variable, or to specify the absolute path to the commands in the body of the script.