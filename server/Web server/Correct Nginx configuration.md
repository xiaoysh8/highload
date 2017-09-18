Correct Nginx configuration

Stumbling upon the pitfalls in the configuration and operation of the web server is very easy. But it is difficult to understand the cause of incorrect or not always correct / erroneous work, if all the rules are observed.

Root inside the location section

There is nothing wrong with placing the root directory inside the location . But if the location does not match, then it will not have access to the root directory. It is more correct to do so:

server {
    server_name www.somesite.com;
    root /var/www/nginx-default/;
    location / {
        # [...]
    }
    location /foo {
        # [...]
    }
    location /bar {
        # [...]
    }
}
# Specifying root inside the server section

Multiple directives index

You do not need to produce a large number of index directives . Write it once in the http :

http {
    index index.php index.htm index.html;
    server {
        server_name www.somesite.com;
        location / {
            # [...]
        }
    }
    server {
        server_name somesite.com;
        location / {
            # [...]
        }
        location /foo {
            # [...]
        }
    }
}
# Index will be automatically inherited in all sections

Using if

Are you aware that if = evil ? When using the directive you need to be careful, it's easy to make a mistake. So, if possible, avoid using if.

Server name

Suppose your site is on the somesite.com domain and you redirect to it users who go to www.somesite.com:

server {
    server_name somesite.com *.somesite.com;
        if ($host ~* ^www\.(.+)) {
            set $raw_domain $1;
            rewrite ^/(.*)$ $raw_domain/$1 permanent;
        }
        # [...]
    }
}
# Checks and redirects the host

Here are a few problems. Home - if. Regardless of the host request (with or without www), Nginx will still check if. For each request. In return, you can do this:

server {
    server_name www.somesite.com;
    return 301 $scheme://somesite.com$request_uri;
}
server {
    server_name somesite.com;
    # [...]
}
# Use $ scheme, which is suitable for http and https

Checking for a file

Do not use if to check for a file:

server {
    root /var/www/somesite.com;
    location / {
        if (!-f $request_filename) {
            break;
        }
    }
}
# The approach is at least not effective

In return, Nginx has a try_files directive :

server {
    root /var/www/somesite.com;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
# Sequence checking for a file, if it does not exist, then sends to index.html

It is noteworthy that the directive can also be used to protect a web server from unauthorized access .

Sending requests to PHP

If Nginx redirects all requests ending in .php , directly to the PHP interpreter, then the attacker will be able to execute third-party code. PHP by default tries to guess where the wrong request should go. So first of all it is necessary to correct php.ini , specifying:

cgi.fix_pathinfo=0
# The interpreter will only process the correct queries

Note the correct configuration of Nginx:

 # Перенаправляет запросы только для указанных файлов
location ~* (file_a|file_b|file_c)\.php$ {
    fastcgi_pass backend;
    # [...]
}

 # Отключить выполнение скриптов в пользовательских загрузках
location /uploaddir {
    location ~ \.php$ {return 403;}
    # [...]
}

 # Фильтрация при помощи try_files
location ~* \.php$ {
    try_files $uri =404;
    fastcgi_pass backend;
    # [...]
}

 # Использует вложенное расположение для фильтрации
location ~* \.php$ {
    location ~ \..*/.*\.php$ {return 404;}
    fastcgi_pass backend;
    # [...]
}
# Options can be combined

FastCGI path

Incorrect indication of the ways of placing the scripts FastCGI part leads to a "Primary script unknown" error , which is easily solved .

Rewrite (rewrite)

Use the $ request_uri variable to change the request URI:

rewrite ^ http://somesite.com$request_uri? permanent;

# Или так
return 301 http://somesite.com$request_uri;
# Redirects to page 301

Missing http: //

Add absent http: // very simple:

rewrite ^ http://somesite.com permanent;
# Automatically complements the query

Proxying

Do not redirect all requests to PHP like this:

server {
    server_name _;
    root /var/www/site;
    location / {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
# Send everything to phpcgi.socket

Use the same try_files directive :

server {
    server_name _;
    root /var/www/site;
    location / {
        try_files $uri $uri/ @proxy;
    }
    location @proxy {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
# Send only the requested proxy requests

Or so:

server {
    server_name _;
    root /var/www/site;
    location / {
        try_files $uri $uri/ /index.php;
    }
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
# If Nginx can not process the requested URI on its own, it checks the directories for availability, then passes them to the proxy

The most important thing

The main reason for the erroneous operation of the system (not just Nginx) is a mindless copy-paste. Check the configs, test the application before rolling out, read the documentation.