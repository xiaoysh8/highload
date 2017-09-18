Roll-out of a large system

The rollout (or deployment) of new versions of Web applications has a number of difficulties, because It is necessary to quickly and simultaneously execute groups of actions on different servers. The process usually includes updating the code (php) and statics (js / css / images), changing databases and system settings. 

Once upon a time, new versions appeared very rarely (once a year or even less often). Then there were complex and lengthy update processes, and users received a huge package of changes immediately. Such a time-consuming process sometimes destroyed entire businesses. Now the concept of the new version is minimized to the slightest changes. The dynamics of the development of modern applications is huge, and rolling out updates can occur every day.

Therefore, a number of requirements were added to the rolling out process:

Minimum (preferably zero) manual intervention.
Maximum execution speed (seconds or minutes, but not hours or days).
The ability to quickly return to the last working version.
Scalability, and hence the independence of execution speed from the number of servers.
The main components of any large Web system are frontends, backends, databases and special purpose servers (for example, mail or media storage).

Roll-out of front-ends

Fronts usually perform two functions:

Distribute static files to clients (css, js, pictures).
Balance requests to the application and proxy them to backends.
Therefore, to update the frontend, you must download new static files. After that, perform the minification of css / js if necessary.

In practice, this is usually done like this:

Create a duplicate project folder on the server (for example / production_b if the main folder is / production_a).
Update this folder (/ production_b) to the latest version. It is convenient to use version control systems to simplify the update. For example, Git.
Perform the necessary manipulation (minification, gluing, etc.).
Change the destination folder in the Nginx'a configuration and restart the server.
This approach allows you to instantly translate all users to a new version. 

Nginx's configuration

Imagine that we use this configuration of the Web server:

server {
        index index.html;
        root /production_a;
}
Then after rolling out, you need to replace the root directive with a new path. You can use a simple php script:

<?
$config = file_get_contents('site.conf');
$current_version = strpos($config, '/production_a') ? 'a' : 'b';
$new_version = $current_version == 'a' ? 'b' : 'a';
$config = str_replace('/production_' . $current_version, '/production_' . $new_version, $config);
file_put_contents('site.conf', $config);
# Script for sequential switching between folders

After updating and preparing the code it will be enough to call this script. It will change the current folder to the next one. If at the moment the working folder is "production_a", we do the rollout in "production_b" and switch to it. If "production_b", then we do the rollout in "production_a". Thus, the working folder is constantly changing, and the neighboring folder will always contain the previous working version.

After changing the configuration file, update the Nginx settings:

/etc/init.d/nginx reload
Roll-out of backends

Roll-out of backends is similar in many respects to rolling out front-ends. 

On all servers, you need to update the code in a secondary folder (/ production_b). Then switch the Nginx's configuration to the desired folder:

server {
		...
        root /production_a;
        index index.php;

        location ~* \.(php)$ {
        	fastcgi_pass 127.0.0.1:9000;
        	fastcgi_index index.php;
        	include fastcgi_params;
        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    	}
}
# Dedicated folder must be replaced with "production_b"

Warm up applications

PHP uses different operating code caches to save on re-interpreting the files. Therefore, before switching users to a new folder, it is useful to make the cache warm up.

To do this, it is enough to have a separate host in Nginx, for example:

server {
		host lambda.ruhighload.com;
        root /production_b;
        index index.php;

        location ~* \.(php)$ {
        	fastcgi_pass 127.0.0.1:9000;
        	fastcgi_index index.php;
        	include fastcgi_params;
        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    	}
}
# Separate host lambda.ruhighload.com for warming up the cache of the new version

After that, open a special page that will simply connect all the project files (preload.php):

<?
if ( $_GET['key'] != 12345 ) exit;

$it = new RecursiveDirectoryIterator("/production_b");
foreach(new RecursiveIteratorIterator($it) as $file)
{
	if ( pathinfo($file, PATHINFO_EXTENSION) == 'php' ) include $file;
}
# Connect all php project files

The call of this page should be added to the rollout script so that you do not do this with your hands:

wget http://lambda.ruhighload.com/preload.php?key=12345
# Security Key

Database Migrations

Modifying data and its structure (or migration) is the most difficult task when rolling out new versions. Firstly, changing the structure of the data can take quite a long time. Secondly, errors in the rollout can lead to data loss.

Before building a data migration system, you must ensure that the following rules are met:

Avoid changes in the structure of large tables. If possible, use additional tables to store data. For example, instead of having a users table with [name, gender, email, password] columns, you can have two tables:
user_auth [email, password]
user_info [name, gender]
Never use the removal of columns, tables or data in migrations. These operations must be performed by separate procedures under the close supervision of administrators. Migrations should only contain the addition of columns / tables / data.
Do not add index creation to migrations other than logically required (for example, unique keys). Indexes should be created solely under the load profile on the production database. And also under the close supervision of administrators.
Use vertical tables for cases where table columns can often change. For example, the table structure for different product properties:
product_id
property_name
property_value
This structure will not need to be changed to add a new property to the product.
Technically, migrations are usually organized as a set of files containing SQL queries collected in a separate folder:

# ls highloadcomua / data / migrations /	    
15.comments.add.post_id.sql		    
16.comments.add.content.sql		    
17.comments.add.user_id.sql		    
18.tags.add.titl.sql			    
...
The migration process is performed using a separate server. It updates the folder with migrations (using the version control system). After that, SQL-queries are executed, which appeared in the new migration files. 

An example of a script that detects new files after the update and migrates from them:

<?

# Получаем список выполненных миграций
$executed = file('executed.migrations');


# Получаем список всех миграций
$files = glob('data/migrations/*');
foreach ( $files as $file )
{
	if ( in_array($file, $executed) ) continue;

	# Выполняем миграцию
	exec('mysql database -u root -p12345 < ' . $file, $o, $r);

	# Если нет ошибки, помечаем миграцию, как выполненную
	if ( !$r ) $executed[] = $file;
}

file_put_contents('executed.migrations', implode("\n", $executed));
Updating configurations

On special purpose servers (for example, mail servers), the need to make changes is less frequent than on application servers. However, sometimes you have to make changes in the configurations. For such purposes it is convenient to store all the configurations in the repository. Then, to change the settings, it will be enough to update the configuration files on all servers. 

For example, to update the configuration of the mail server, you could use approximately such a script:

cd /etc/exim4
git pull
update-exim4.conf
/etc/init.d/exim4 restart
For more complex configuration management tasks, it's better to use Chef .

Parallel rollout

In the event that several dozens or more servers are present in the system, serial updating of the code on all servers can take a long time:

for ip in `cat servers.list`; do
    echo "Updating $ip..."
    ssh $ip 'git -C /production_b pull'
done
# Sequential code update

With background processes, you can run all of these commands in parallel:

for ip in `cat servers.list`; do
    echo "Updating $ip..."
    ssh $ip 'git -C /production_b pull' >> /var/log/deploy.log &
done
# Parallel code update on all servers in the background

Return to working version

Rolling out can contain mistakes or simply be unsuccessful in terms of business. In any case, it is always necessary to be able to quickly restore the previous version of the system.

With all the nodes in the described process, this is very easy to do. It is enough to change the working directory of the Web server from the current (for example, production_b) to the previous one (production_a).

For databases it is very important to comply with the rule - the absence of deleting data of any kind in migrations. Then returning to the previous version does not require a reverse structure change. Removing obsolete columns, tables and data should be planned with a delay (for example, not earlier than a week). This will give you enough time for a possible return.

Part of the audience

Major changes (for example, new functions or significant changes in current ones) can have negative consequences after rolling out. This may be due to an unsuccessful business or technical solution. In addition, in a real environment, there are always certain factors that can not be repeated in the development and testing environment (for example, a large number of online users). This means that rolling out big changes always contains a risk.

Therefore, it is convenient to roll out large and important changes in such a way that they are accessible only to a part of the audience. For example, only for one percent of all users. In this case, negative consequences will be limited to only a small part of the audience. 

In practice, this can be achieved by setting a cookie to selected users. For example, let's set each hundredth user to the "tester" cookie:

<?
if ( session::get('id') % 100 == 1 ) setcookie('tester', 1);
...
Then in Nginx you can use this value to send users with this cookie to another (new) folder:

server {
		server_name ruhighload.com;

        set $rt '/production_a';

        if ($http_cookie ~* " tester=1") {
            set $rt '/production_b';
        }

        root $rt;

        location ~* \.(php)$ {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;
                include fastcgi_params;

                fastcgi_param SCRIPT_FILENAME $rt$fastcgi_script_name;
        }
}
# If the tester is installed, change the variable $ rt to another path

Validation of rollout

The last step in the rollout process is to check the availability of the application. This is to make sure that there are no critical errors. In the simplest case, this can be a test page that contains checks for the most important components:

<?

# проверяем подключение к базе данных
$time = mysql::col('SELET NOW()');
if ( !$time ) echo 'error getting time from mysql';


# проверяем код 200 от основных страниц
$pages = ['/', '/speed', '/server'];
foreach ( $pages as $page )
{
	$c = curl_init('http://ruhighload.com' . $page);
	$result = curl_exec($c);
	$status = curl_getinfo($c, CURLINFO_HTTP_CODE);
	curl_close($c);
	if ( $status != 200 ) echo 'error on page ' . $page;
}


# еще можно проверить php_error.log на наличие ошибок
# еще можно проверить php-fpm.slow-log на появление медленных скриптов

# и т.п.
# hearbeat.php - simple script for quick checking of important site components

In the presence of unit-tests, it makes sense to use their part for the validation of the rollout. In the case of detecting errors, it is best to automatically roll back to the previous version, then repair the malfunction.

The most important

The most important is the description of the general rolling process:

Updating files in a specially allocated folder (/ production_b) using the version control system (Git, SVN, etc.).
Preliminary preparation (minification, warming up caches, data migration).
Switching the audience or its parts to the new version (changing the path from / production_a to / production_b on the Web server).
Check the main components of the system (for example, the HTTP response code from the main pages) and return to the previous version if errors are detected.
The subsystem of rollout is the same dynamic component of the application as any other. It constantly needs to be improved and improved. A good rule of thumb is to have an autonomous rollout system that does not require manual operations .

Do not rush to use sophisticated hedging control systems with a bunch of integrations in just about everything. Own solutions are often much simpler, so it's easier to manage and more flexible to use.

In spite of maximum automation, any rollout must always occur under the control of administrators or developers. Never do blind rollouts, always wait for "Deployment finished successfully" from your system.