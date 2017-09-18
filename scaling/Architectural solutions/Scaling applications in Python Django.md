Scaling applications in Python / Django

For more than 20 years of existence, Python has got a crowd of fans, a lot of modules for all popular platforms and a lot of frameworks. Among the latter deserved popularity is enjoyed by Django, which is used in Instagram, Disqus, Mozilla, Pinterest. Their experience shows that Django copes with the highest load and is suitable for scaling projects. Django app

Where to begin?

You need to start with profiling , analyzing the load on the server and checking the web application for load . Also, read the principles of building highly loaded applications and do not forget to do client optimization .

Caching

After all of the above, you can start to refine the Django-application. Start by caching the database. Django cache

Django has its own framework for caching . You must create a separate database table for the cache:

python manage.py createcachetable cache_table
# Creates a cache_table

Then you need to add the configuration file settings.py :

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'cache_table',
    }
}
# Thus, it is easy to connect a Memcached module for Python

Now you can cache the data you need:

from django.views.decorators.cache import cache_page

@cache_page(60 * 60)
def product_detail(request, product_id):
    ...
# Caches the page for 60 minutes (60 seconds * 60 minutes)

The caching system of Django uses the full address of the request ( "https://www.example.com/stories/2005/?order_by=author" ) to create the keys. But if you need to give different content depending on the cookie, language, user agent, the framework can vary the return using the headers Vary :

from django.views.decorators.vary import vary_on_headers

@vary_on_headers('User-Agent', 'Cookie')
def my_view(request):
    ...
# The system will create separate caches for different users

Do not forget that you can cache pages with dynamic content . Use tricks. New comments are not required to be shown at the time of their publication, the delay in displaying in a few minutes is not critical. Note that even caching for 10 seconds will reduce the load on the entire system. To improve the overall performance of the web project, you can use caching with ESI , Varnish , and Memcache is also useful .

Managing static content

In most cases, it's better not to use Django to distribute statics (except authorization data). But the framework is convenient to use for managing static files, which will then be returned by the web server.

First you need to edit the configuration file settings.py :

STATIC_ROOT = os.path.join(BASE_DIR, "static")
# Specifies that static files will be stored in the main directory of the web application

After you can run the static builder:

python manage.py collectstatic
# Collects static files in the specified directory

When Python completes the build process, the static directory appears in the main application directory , which will collect all the files (CSS, JS, pictures, etc.), parsed by subdirectories.

It remains to specify in which directories these files are located:

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
# Specifying a directory with a static in the settings.py file

Better yet, specify the path in the settings of the web server, which will perform the work faster than Django. For Nginx, you need to add:

server {
    ...
    location /static/ {
        alias /srv/testapp/static/;
    }
}
# There may be several such directories for different application categories

uWSGI

WSGI is the protocol of Python interaction with the web server. And uWSGI is a modern implementation of the protocol. That is, the user sends a request to the web server, which in turn through uWSGI passes it to the Django application and receives a response.Django uWSGI

Using uWSGI, you can limit the number of running application processes, usually one parent process and two vendors are running. If necessary, you can increase the number of vorkers, or reduce their number.

Setting up and enabling

uWSGI will allow the application Django and Nginx to communicate. First you need to install it:

sudo aptitude install uswgi
# Installation through system packages

The module can also be installed via pip or build from sources.

After installation, two new directories appear: / etc / uwsgi / apps-available and / etc / uwsgi / apps-enabled . The scheme of operation is simple, for the application to work, its configuration file must be put in apps-available and referenced in apps-enabled .

The project settings file project testapp.ini will look like this:

[uwsgi]
touch-reload = /tmp/testapp
socket = 127.0.0.1:9001
workers = 2
chdir = /srv/testapp
env = DJANGO_SETTINGS_MODULE=testapp.settings
module = django.core.handlers.wsgi:WSGIHander()
# Specifies the socket and the number of vorkers

It remains to create an empty file specified in the touch-reload :

touch /tmp/testapp
# uWSGI will reboot when the file is changed

And to connect projects:

sudo ln -s /etc/uwsgi/apps-available/testapp.ini /etc/uwsgi/apps-enabled/testapp.ini
# Do not forget to specify your own paths

Now you need to reboot uWSGI and configure the web server.

Configuring Nginx

The web server needs to be rebuilt by adding the ngx_http_uwsgi_module module . Then you need to create a configuration file for the / etc / nginx / sites-available / testapp web application that looks like:

server {
        listen          80;
        server_name     $hostname;
        access_log /srv/testapp/logs/access.log;
        error_log /srv/testapp/logs/error.log;

        location / {
            uwsgi_pass      127.0.0.1:9001;
            include         uwsgi_params;
            uwsgi_param     UWSGI_SCHEME $scheme;
            
        }

        location /static {
            alias /srv/testapp/static/;
            index  index.html index.htm;

        }
}
# Specifying the socket and address with static files

You can also configure the cluster using Nginx as the balancer :

upstream uwsgicluster {
     server 127.0.0.1:9001;
     server 192.168.100.101:9001;
     server 192.168.100.102:9001;
     server 192.168.100.103:9001;
     server 192.168.100.104:9001;
}
# Nginx will distribute the load evenly

The most important thing

Caching and the uWSGI module can improve the performance and speed of the Django web application. This architecture is easy to scale and does not require additional investments. Do not forget to optimize the settings of the web server to improve the responsiveness of the site .