It's very easy to stumble on the pitfalls in the configuration and operation of the web server. But it is difficult to understand the reason for the incorrect or not always correct operation if all the rules are respected.

Root inside location

There is nothing wrong in placing root-directory inside the location. But if the location does not match, then it will not have access to the root directory. Correct way:

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
# Specify root inside server

Several index directives

You do not need to produce a large number of index directives. Specify it once in http block:

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

You know that if is evil, right? When using directive you need to be careful, it's easy to make a mistake. Try to avoid using if.

Server name

Suppose your site is on somesite.com domain and you redirect users, that go to www.somesite.com:

server {
    server_name somesite.com *.somesite.com;
        if ($host ~* ^www\.(.+)) {
            set $raw_domain $1;
            rewrite ^/(.*)$ $raw_domain/$1 permanent;
        }
        # [...]
    }
}
# Checks and redirects host

There are several problems, and the main if. Regardless of the host request (with or without the www), Nginx still checks if. For each request. Instead, you can do so:

server {
    server_name www.somesite.com;
    return 301 $scheme://somesite.com$request_uri;
}
server {
    server_name somesite.com;
    # [...]
}
# Used $scheme, which is suitable for http and https

Check for file

Do not use if to check for the file:

server {
    root /var/www/somesite.com;
    location / {
        if (!-f $request_filename) {
            break;
        }
    }
}
# The approach is at least ineffective

Instead, there is a directive try_files:

server {
    root /var/www/somesite.com;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
# Checks the sequence for the file, if there is no, go to the index.html

It is noteworthy that the directive can also be used to protect Web servers from unauthorized access.

Redirect requests to PHP

If Nginx redirects all requests ending in .php, directly to the PHP interpreter, there are opportunities for the attackers for arbitrary code execution. By default PHP tryes to guess where improper request should lead. So you first need to modify php.ini, stating:

cgi.fix_pathinfo=0
# The interpreter will handle only valid requests

Here's the correct Nginx configuration:

 # Redirects to PHP only for specified files
location ~* (file_a|file_b|file_c)\.php$ {
    fastcgi_pass backend;
    # [...]
}

 # Forbidden execution from user upload
location /uploaddir {
    location ~ \.php$ {return 403;}
    # [...]
}

 # Filter with try_files
location ~* \.php$ {
    try_files $uri =404;
    fastcgi_pass backend;
    # [...]
}

 #Uses inserted location for filtering
location ~* \.php$ {
    location ~ \..*/.*\.php$ {return 404;}
    fastcgi_pass backend;
    # [...]
}
# Parameters can be combined

FastCGI path

A wrong FastCGI script path often leads to error "Primary script unknown", which can be easily solved.

Rewrite

Use $request_uri variable to change the Request URI:

rewrite ^ http://somesite.com$request_uri? permanent;

# or
return 301 http://somesite.com$request_uri;
# Redirects with error 301

Missing http://

Addition of missing http:// is very simple: code>rewrite ^ http://somesite.com permanent;

# Automatically supplements the request

Proxying

Do not redirect all requests to PHP in this form:

server {
    server_name _;
    root /var/www/site;
    location / {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
# Transmits all on phpfcgi.socket

Use the try_files instead:

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
# Transfers only needed requests

Or like this:

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
# If Nginx can not process the request URI himself, then checks the directory existance, and then transmits to the proxy

The most important

The main cause of erroneous operation of the system (not just Nginx) — mindless copy-paste. Check your config files, test the application before deployment, read the documentation.