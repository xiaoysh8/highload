NginScript in practice

NginScript is the JavaScript runtime environment in Nginx. Its parser currently supports ECMAScript 5 (they promise to extend support in the future). There is an internal bytecode compiler, which is then executed in Nginx every time JavaScript is called. This is done using a virtual machine based on registers. 

![](https://i.onthe.io/shpzkl57dpkt5qjdc.fa574aac.jpg)

nginScript VM is slightly different from popular virtual machines for JavaScript, because it focuses on server requirements, not browser optimization. Interestingly, each process runs in a separate virtual environment, so you do not need a garbage collector.

The Nginx configuration language has been supplemented with the syntax, by means of which you can embed the code blocks on the JS directly into the configuration file. For this, two new directives are used: js_set and js_run . They are executed as the HTTP transactions are processed, and for each request, you can adjust the settings of the web server, correct the request or response, and implement very complex conditions.

In addition, you can pause and resume any blocking operations, HTTP subqueries, for example. NginScript is also useful for creating stubs as a temporary (and not very) solution to the problem. Several other important functions of the function: blocking bad / malicious requests or vulnerabilities, limiting the intensity of requests.

Installing the nginScript module

To add a module, you will need Nginx latest version:

wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar -xzvf nginx-1.9.9.tar.gz
# Download and unzip Nginx from the repository, requires installed Mercurial

Next, you need to get the sources of nginScript development:

hg clone http://hg.nginx.org/njs
# Cloning a repository with nginScript from Mercurial

Then you need to rebuild and update (install) Nginx:

cd nginx-1.9.9
./configure --add-module=../njs/nginx --prefix=/your/custom/installation/directory
make
make install
# Compiling and installing Nginx with the nginScript module

Note that nginScript is still in active development, so it's worth keeping an eye on the change log and updated in time to current and more stable versions.

js_set

The directive gives the value of the variable to the result of executing the JS code. nginScript 

![](https://i.onthe.io/shpzkl3mqpbeib0i8g.9f47013f.jpg)

A variable assigned with js_set can be used in any Nginx directive that accepts variables: limit_req_zone , proxy_pass and sub_filter , for example.

Examples of using js_set

nginScript provides a query object and is able to pass it to the function as the first parameter. You can read and set the properties of this object and use the methods for accessing and modifying the query.

js_set $summary "function summary(req, res) {
    var a, s, h;

    s = 'Request summary\n\n';

    s += 'Method: ' + req.method + '\n';
    s += 'HTTP version: ' + req.httpVersion + '\n';
    s += 'Host: ' + req.headers.host + '\n';
    s += 'Remote Address: ' + req.remoteAddress + '\n';
    s += 'URI: ' + req.uri + '\n';

    s += 'Headers:\n';
    for (h in req.headers) {
        s += '  header \"' + h + '\" is \"' + req.headers[h] + '\"\n';
    }

    s += 'Args:\n';
    for (a in req.args) {
        s += '  arg \"' + a + '\" is \"' + req.args[a] + '\"\n';
    }

    return s;
}";

http {
    js_set $summary "
    var a, s, h;

    s = 'JS summary\n\n';

    s += 'Method: ' + $r.method + '\n';
    s += 'HTTP version: ' + $r.httpVersion + '\n';
    s += 'Host: ' + $r.headers.host + '\n';
    s += 'Remote Address: ' + $r.remoteAddress + '\n';
    s += 'URI: ' + $r.uri + '\n';

    s += 'Headers:\n';
    for (h in $r.headers) {
        s += '  header \"' + h + '\" is \"' + $r.headers[h] + '\"\n';
    }

    s += 'Args:\n';
    for (a in $r.args) {
        s += '  arg \"' + a + '\" is \"' + $r.args[a] + '\"\n';
    }

    s;
    ";

    server {
    listen 8000;

    location /summary {
        return 200 $summary;
    }
}
# Return query parameters back to the client

With nginScript, you can redirect traffic based on query data: cookies, arguments, headers, or keywords in the body of the request.

upstream my_upstream0 {
    server server1.example.com;
    server server2.example.com;
}
upstream my_upstream1 {
    server server3.example.com;
    server server4.example.com;
}

js_set $my_upstream "
    var s, upstream, upstream_num;

    upstream = $r.args.upstream;

    // преобразование числа апстримов в целое число
    upstream_num = +upstream | 0;

    if (upstream_num < 0 || upstream_num > 1) {
        upstream_num = 0;
    }

    s = 'my_upstream' + upstream_num;

    s;
";

server {
    listen 80;

    location / {
        proxy_set_header Host $host;
        proxy_pass http://$my_upstream;
    }
}
# Routing traffic based on the upstream argument

js_run

The js_run directive runs at the content generation stage and is used to execute JavaScript and create an HTTP response. nginScript js_run

Example of using js_run

It is located inside the location block in the Nginx configuration file and runs the execution of the JS code when the location matches the URL request.

location / {
    js_run "
        var res;
        res = $r.response;
        res.status = 200;
        res.send('js_run test');
        res.finish();
    ";
}
# Displaying the text "js_run test"

The most important thing

The new nginScript module is not intended to replace the Lua module or other Nginx built-in languages. But with it you can test configs, check functionality, create fixes and close holes. In fact, nginScript allows you to refactor the process of developing and deploying web applications.