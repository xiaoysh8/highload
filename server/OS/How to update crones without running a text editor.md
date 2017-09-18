How to update crones without running a text editor

Everyone knows that you can do this:

$ crontab -e
After executing the command, the favorite nano / vi / mcedit starts, in which you can edit the schedule, and after the editor is closed, the schedule file will be updated.

But there are times when schedules need to be updated automatically using scripts, for example, in conditions of continuous integration or projects in which the schedule is set by the user.

Use /etc/cron.d

Such schedules can be decomposed into files for which there is a special folder:

$ ls /etc/cron.d
php
In this folder you can add your own scripts, breaking them for convenience into files (there can be many files). The only difference between the scripts in the file is that before the command you need to specify the user, on behalf of which this command will be executed, for example:

*/10 * * * * www-data /var/www/task/flush_cache.sh
With the help of this approach, the crontab schedule can be stored in the project code and when you update, update the crontab automatically by running one simple command:

cat /var/www/my_project/conf/crontab > /etc/cron.d/my_project
Important to remember

Do not forget that at the end of each crontab there must be an empty string, otherwise the schedule will not start