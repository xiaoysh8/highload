Using Lua in Nginx

Openresty, a large set of modules for Nginx, opens many opportunities for development directly on the popular Web server. One of the main advantages of this package is the extension to support the Lua language in Nginx.

Installation

In Debian systems, all you need is in the package:

apt-get install nginx-extras
Hello world

We derive hello world :

server {
  location /hello {
    default_type 'text/plain';

    content_by_lua '
        ngx.say("Hello world!")
    ';
  }
}
# We display the known string directly with Nginx'a

Now restart Nginx :

nginx -s reload
And at http: // site / hello we will see:

Hello world!
HTML output

To output HTML, it's enough to replace the content type and specify the content itself:

server {
  location /hello {
    default_type 'text/html';

    content_by_lua '
        ngx.say("Hello <b>world</b>!")
    ';
  }
}
# Display HTML from Nginx Lua

Organization of the code

For convenience, it is worth using external Lua files:

server {
  location / {
    default_type 'text/plain';
    content_by_lua_file /var/www/lua/index.lua;

    # Отключим кэширование кода для разработки
    # (это нужно закомментировать, когда выкатим на продакшн)
    lua_code_cache off;
  }
}
# Lua code download from external files

During development, it is convenient to use lua_code_cache , because the file code can be changed without restarting Nginx.

Multiple handlers

server {
  location / {
    default_type 'text/plain';
    content_by_lua_file /var/www/lua/index.lua;
  }

  location /admin {
    default_type 'text/plain';
    content_by_lua_file /var/www/lua/admin.lua;
  }
}
Global Variables

For settings and statistics it is convenient to use global variables (they will have the same values ​​for all queries):

http {
    # объявляем глобальный контейнер
    lua_shared_dict stats 1m;

    server {
        location / {
            content_by_lua '
		# увеличим переменную hits на 1 при каждом запросе
                ngx.shared.stats:incr("hits", 1)
		
		# выведем текущее значение
                ngx.say(ngx.shared.stats:get("hits"))
            ';
        }
    }
}
# Use the global variable to count the number of requests

Working with data

Nginx supports work with different databases, including. Mysql and Redis.

apt-get install lua-nginx-redis
Example of a simple script for counting the number of requests in Redis:

server {
        location / {
            content_by_lua '
		local redis = require "nginx.redis"
		local red = redis:new()
		local ok, err = red:connect("127.0.0.1", 6379)
		ok, err = red:incr("test")
		local res, err = red:get("test")
		ngx.say("hits: ", res)
            ';
        }
}
# Increase the test counter with Redis

The most important thing

Openresty allows you to use Nginx not just as a Web server, but as a full-fledged platform. With Lua, you can implement a large set of functions, including and work with data.