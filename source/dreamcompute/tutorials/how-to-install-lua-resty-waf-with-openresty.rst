========================================================
How to install OpenResty + lua-resty-waf on Ubuntu 14.04
========================================================

OpenResty is a software bundle containing the Nginx web server,
`lua-nginx-module <https://github.com/openresty/lua-nginx-module>`__, the
LuaJIT compiler, and a number of Lua modules designed to extend the capability
of Nginx and transform it into a full-fledged application server.
`lua-resty-waf <https://github.com/p0pr0ck5/lua-resty-waf>`__ is a high
performance web application firewall (WAF) written for the OpenResty stack,
leveraging the scalable architecture of Nginx, while providing a ModSecurity-compatible
rule syntax. This allows users to move their ModSecurity WAF installations to the
OpenResty ecosystem.

This tutorial walks you through the installation process for OpenResty and lua-resty-waf
on a DreamCompute instance running Ubuntu 14.04.

Installing prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~


You must compile OpenResty from source on Ubuntu distributions. First, update
the system package repository listings and install the following packages:

.. code-block:: console

    [root@server]# apt-get update && apt-get -y install make gcc libssl-dev g++ \
        liblua5.1-0-dev python-minimal python2.7 libjson-perl git

Preparing the source
~~~~~~~~~~~~~~~~~~~~

Download and unpack the OpenResty source in a local directory:

.. code-block:: console

    [root@server]# cd /usr/local/src
    [root@server]# wget https://openresty.org/download/openresty-1.11.2.1.tar.gz && \
        tar -zxf openresty-1.11.2.1.tar.gz

Most web application firewalls use a number of pattern matching techniques when
examining HTTP traffic, including regular expressions. Because complex regular
expressions can be expensive to process, it makes sense to use a regex library
that is capable of optimizing regex execution. Modern versions of the Perl
Compatible Regular Expressions (PCRE) library, the regular expression library
used by Nginx and OpenResty, are capable of performing just-in-time compilation
of regular expressions which greatly improves regex matching performance.

Because the version of PCRE provided by the default Ubuntu package does not contain
JIT support, you must download and unpack the PCRE source:

.. code-block:: console

    [root@server]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.bz2 \
        && tar -jxf pcre-8.39.tar.bz2

Note: Compiling and installing PCRE is not necessary, as it is statically compiled
into OpenResty.

Compiling OpenResty
~~~~~~~~~~~~~~~~~~~

Build the OpenResty bundle from source and install it to the system:

.. code-block:: console

    [root@server]# cd openresty-1.11.2.1 && ./configure --with-debug \
        --with-pcre=/usr/local/src/pcre-8.39 \
        --with-pcre-jit \
        --with-pcre-opt=-g && \
        make && \
        make install

Nginx is built and configured to use the previously-downloaded version of PCRE
that supports JIT, instead of the system PCRE, which greatly improves performance.

.. Note:: **`--with-debug`** and **`--with-pcre-opt=-g`** are provided
        to allow Nginx to write debugging information when configured to do
        so. Adding these options causes performance degradation in
        high-concurrency environments, so do not enable these options for
        high-traffic sites.

Testing OpenResty
~~~~~~~~~~~~~~~~~

Once OpenResty is installed, you can configure a simple test to confirm that it is
responding as expected.

To test, add the following configuration snippet inside the existing `server` block
located in the configuration file:

.. code-block:: console

    [root@server]# cd /usr/local/openresty/nginx
    [root@server]# vi conf/nginx.conf

.. code:: nginx

    location /foo {
        content_by_lua_block {
            ngx.say("Hello, world!")
        }
    }

Once it completes, start Nginx:

.. code-block:: console

    [root@server]# ./sbin/nginx

Send a test request to the test location block:

.. code-block:: console

    [root@server]# curl http://<instance-ip>/foo
    Hello, world!

Building lua-resty-waf
~~~~~~~~~~~~~~~~~~~~~~

Once OpenResty is installed and working, download and install lua-resty-waf. The
source for lua-resty-waf lives in a GitHub repo, so clone the repo to a local
source, and then make and install the project:

.. code-block:: console

    [root@server]# cd /usr/local/src
    [root@server]# git clone --recursive https://github.com/p0pr0ck5/lua-resty-waf.git && \
        cd lua-resty-waf && \
        make && \
        make install

Configuring lua-resty-waf
~~~~~~~~~~~~~~~~~~~~~~~~~

After installing lua-resty-waf, return to the Nginx config file and add the
basic directives to run lua-resty-waf:

.. code-block:: console

    [root@server]# cd /usr/local/openresty/nginx
    [root@server]# vi conf/nginx.conf

Add the following directive to the `http` block, above the existing `server`
block:

.. code:: nginx

    init_by_lua_block {
        require "resty.core"
        local waf = require "resty.waf"
        waf.init()
    }

Add the following directives to the test `location` directive created earlier:

.. code:: nginx

    access_by_lua_block {
        local lrw = require "resty.waf"
        local waf = lrw:new()
        waf:set_option("debug", true)
        waf:set_option("mode", "ACTIVE")
        waf:exec()
    }

    log_by_lua_block {
        local lrw = require "resty.waf"
        local waf = lrw:new()
        waf:write_log_events()
    }

These directives instruct OpenResty to execute lua-resty-waf when a request
is handled by the test location directive, and to deny requests that look
malicious. lua-resty-waf ships with a basic set of rules that mimic the
`OWASP CRS <https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project>`__,
which provides protection against HTTP protocol anomalies, known suspicious user
agents, cross-site scripting (XSS), and SQL injection (SQLi) attacks.

To test, reload Nginx and send the following request:

.. code-block:: console

    [root@server]# ./sbin/nginx -s reload
    [root@server]# curl 'http://<instance-ip>/foo?a=alert(1)'

Nginx should return a 403 Forbidden response, instead of the 200 OK and
'Hello, world!' received earlier.

Further configuration
~~~~~~~~~~~~~~~~~~~~~

By default, lua-resty-waf logs transactions that it blocks to the Nginx
error log. This can be difficult to parse out, especially with debug logging
enabled.

You can configure lua-resty-waf to write event logs to a file on disk, which provides
more detailed information about the request, by adding the following
directives to the previously created `access_by_lua_block`, above the
`waf:exec()` directive:

.. code:: nginx

    waf:set_option("event_log_target", "file")
    waf:set_option("event_log_target_path", "/tmp/waf.log")
    waf:set_option("event_log_request_headers", true)
    waf:set_option("event_log_request_arguments", true)
    waf:set_option("event_log_request_body", true)
    waf:set_option("event_log_periodic_flush", 1)

Reload Nginx, and then send the test bad request again:

.. code-block:: console

    [root@server]# ./sbin/nginx -s reload
    [root@server]# curl 'http://<instance-ip>/foo?a=alert(1)'

lua-resty-waf creates the event log file and populates it with a JSON
entry containing details about the request. JSON that is not pretty-printed can
be hard to eyeball; instead, use the following snippet to clean up the log entry:

.. code-block:: console

    [root@server]# perl -e '
        use JSON;
        print to_json(from_json(<>), { pretty => 1, canonical => 1 });
    ' < /tmp/waf.log

Further exploration
~~~~~~~~~~~~~~~~~~~

Besides basic request protection, lua-resty-waf can fulfill a wide variety of
needs in a WAF installation, including:

- Analyze any aspect of an HTTP request or response for anomalous behaviors
- Mitigate brute-force attacks to any request resource
- Use real-time DNS blacklists to deny known malicious hosts
- Send audit event logs to a remote TCP/UDP/syslog server
- Use memcached or redis to store long-term variables

Check out the `lua-resty-waf Readme
<https://github.com/p0pr0ck5/lua-resty-waf/blob/master/README.md>`__
and `wiki <https://github.com/p0pr0ck5/lua-resty-waf/wiki>`__ for
updates on the project and further tutorials on specific behaviors.
There is also a `#lua-resty-waf` channel on Freenode IRC.

.. meta::
    :labels: nginx openresty security
