Profiling and monitoring are essential for the stable operation of highly loaded systems. In addition to logging errors and queries, Nginx has its own statistics module ngx_http_stub_status_module with the basic data on the web server operation. Nginx metrics

The module is not included into system by default, so you'll will have to recompile Nginx with --with-http_stub_status_module.

Then it is necessary to include the statistics in the configuration file /etc/nginx/nginx.conf, adding the following parameters in the server section:

location /nginx_status {
  stub_status on;
  access_log off;
  allow 127.0.0.1;
  deny all;
}
# Allows access only locally

Note that if you want to see the metrics from the Internet, then you need to enter your IP.

When connecting at http://somesite.com/nginx_status following statistics will be available:

Active connections;
Accepts;
Handled;
Requests;
Reading;
Writing;
Waiting.
Munin to collect statistics

Built-in Nginx statistics suits one-time problem analysis. So for the continuous monitoring of data, this module is better to use in tandem with the monitoring system, such as Munin.

After installation and setup, Munin should be connected to the Nginx. To do this, you must edit the Web server configuration file and add:

location /munin {
        alias   /var/www/html/munin/;
        allow 127.0.0.1;
                allow XX.XX.XX.XX;
        deny all;
        index  index.php index.html index.htm;
        location ~* \.(png|jpg|jpeg|gif|ico)$ {
        }
}
# Specify your IP to access from outer world

Or you can create a separate /etc/nginx/conf.d/munin.conf configuration file with the following content:

listen 80 default_server;
access_log off;
server_name _; 
server_name_in_redirect off;
root  /var/www/html;

location /munin {
        alias   /var/www/html/munin/;
        allow 127.0.0.1;
                allow XX.XX.XX.XX;
        deny all;
        index  index.php index.html index.htm;
        location ~* \.(png|jpg|jpeg|gif|ico)$ {
        }
}

location /nginx_status {
        stub_status on;
        access_log   off;
        allow 127.0.0.1;
        deny all;
}
}
# In this case, you can remove nginx_status from the main configuration file

Do not forget to restart Nginx and verify the configuration. Then you can check that Munin sees a web server:

# munin-node-configure --suggest | grep nginx
nginx_request | no | yes
nginx_status | no | yes
# Shows that plugins are disabled, but the statistics module is connected

Now connect the plugins to draw graphs for Nginx metrics:

ln -s /usr/share/munin/plugins/nginx_request /etc/munin/plugins/ -v
ln -s /usr/share/munin/plugins/nginx_status /etc/munin/plugins/ -v
# Create symbolic links to the plugins

Built in Munin plugins provide information about the server status and the number of requests. Additionally, you can connect the unofficial plugins to extend the functionality:

wget https://github.com/perusio/nginx-munin/archive/master.zip
unzip master.zip
cd nginx-munin-master/
cp nginx_connection_request nginx_memory /etc/munin/plugins/ -fv
# Download and deployment of additional plugins for monitoring connections/queries and RAM

Don't forget to restart munin-node

Graphics

Munin displays statistics in graphs that help you quickly assess the extent of the problem. For example, you need to check the server configuration, if you notice a large number of HTTP errors: HTTP errors

And this graph suggests that it is necessary to check the operation of the caching system: 
Nginx Cache hit rate

Monitoring with Zabbix

For parsing Nginx statistics, you can also use Zabbix. If you are already using Zabbix, then you only need to connect the web server metrics. First, create a directory for the script and put in it a parsing script:

mkdir /etc/zabbix/scripts/

vim /etc/zabbix/scripts/nginx-stats.sh


#!/bin/bash
#<p class="code-comment">;#  OPTIONS VERIFICATION
</p>if [[ -z "$1" || -z "$2" || -z "$3" ]]; then
exit 1
fi

##### PARAMETERS ###<p class="code-comment"># 
</p>RESERVED="$1"
METRIC="$2"
STATSURL="$3"

#
CURL="/usr/bin/curl"
TTLCACHE="55"
FILECACHE="/tmp/zabbix.nginx.`echo $STATSURL | md5sum | cut -d" " -f1`.cache"
TIMENOW=`date '+%s'`

##### RUN ###<p class="code-comment"># 
</p>if [ -s "$FILECACHE" ]; then
TIMECACHE=`stat -c"%Z" "$FILECACHE"`
else
TIMECACHE=0
fi
if [ "$(($TIMENOW - $TIMECACHE))" -gt "$TTLCACHE" ]; then
echo "" >> $FILECACHE # !!!
DATACACHE=`$CURL --insecure -s "$STATSURL"` || exit 1
echo "$DATACACHE" > $FILECACHE # !!!
fi
if [ "$METRIC" = "active" ]; then
cat $FILECACHE | grep "Active connections" | cut -d':' -f2
fi
if [ "$METRIC" = "accepts" ]; then
cat $FILECACHE | sed -n '3p' | cut -d" " -f2
fi
if [ "$METRIC" = "handled" ]; then
cat $FILECACHE | sed -n '3p' | cut -d" " -f3
fi
if [ "$METRIC" = "requests" ]; then
cat $FILECACHE | sed -n '3p' | cut -d" " -f4
fi
if [ "$METRIC" = "reading" ]; then
cat $FILECACHE | grep "Reading" | cut -d':' -f2 | cut -d' ' -f2
fi
if [ "$METRIC" = "writing" ]; then
cat $FILECACHE | grep "Writing" | cut -d':' -f3 | cut -d' ' -f2
fi
if [ "$METRIC" = "waiting" ]; then
cat $FILECACHE | grep "Waiting" | cut -d':' -f4 | cut -d' ' -f2
fi

#
exit 0
# Checking the status and collecting Nginx statistics

Note that the script uses caching to reduce the number of queries when testing metrics.

And do not forget to assign permissions:

chown zabbix:zabbix -R /etc/zabbix/scripts/
chmod 750 /etc/zabbix/scripts/
chmod 700 /etc/zabbix/scripts/nginx-stats.sh
# Access to the scripts directory only from localhost

Last but not least, edit the Zabbix agent /etc/zabbix/zabbix_agentd.conf settings:

UserParameter=nginx[*],/etc/zabbix/scripts/nginx-stats.sh "none" $1 $2
# Specifies a parsing script nginx-stats.sh

After restarting the agent, you need to create a template in the Zabbix frontend to draw relevant graphs: Nginx Zabbix graph

The most important

Monitoring of Nginx using Munin or Zabbix will help quickly respond to problems and to obtain information about the errors in the form of graphs and visual notifications.