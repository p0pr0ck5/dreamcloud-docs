=====================================================
How to Install libmodsecurity + Nginx on Ubuntu 14.04
=====================================================

`ModSecurity <https://www.modsecurity.org/>`_, originally written as a web
application firewall (WAF) for Apache servers, is the de facto standard for
open-source WAF solutions. Recent work on the project has shifted focus toward
providing a generic shared library that any web server can use to protect
HTTP(S) requests. These instructions touch on building and configuring
libmodsecurity for a DreamCompute instance running Ubuntu 14.04.

Building libmodsecurity
~~~~~~~~~~~~~~~~~~~~~~~

First, install the necessary packages and libraries used to build source
projects, as well as libraries used specifically by libmodsecurity:

.. code-block:: console

    [root@server]# apt-get install automake gcc make pkg-config libtool g++ libfl-dev bison \
        build-essential libbison-dev libyajl-dev liblmdb-dev libpcre3-dev \
        libcurl4-openssl-dev libgeoip-dev libxml2-dev


Next, download and unpack the most recent source of libmodsecurity. This is
available from the ModSecurity GitHub project, on the `libmodsecurity` branch:

.. code-block:: console

    [root@server]# git clone https://github.com/SpiderLabs/ModSecurity
    Cloning into 'ModSecurity'...
    remote: Counting objects: 20508, done.
    remote: Compressing objects: 100% (72/72), done.
    remote: Total 20508 (delta 16), reused 0 (delta 0), pack-reused 20435
    Receiving objects: 100% (20508/20508), 33.93 MiB | 9.49 MiB/s, done.
    Resolving deltas: 100% (14572/14572), done.
    Checking connectivity... done.

.. code-block:: console

    [root@server]# cd ModSecurity/
    [root@server]# git checkout -b origin/libmodsecurity
    Switched to a new branch 'origin/libmodsecurity'

Initialize and update the git submodules that libmodsecurity requires:

.. code-block:: console

    [root@server]# git submodule init
    [root@server]# git submodule update

Finally, configure, build, and install the libmodsecurity library:

.. code-block:: console

    [root@server]# ./build.sh && ./configure && make && make install

Building Nginx with libmodsecurity
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that libmodsecurity has been installed and is available to be used by
third-party programs, Nginx can be compiled with the ModSecurity-nginx connector
to load libmodsecurity and process requests.

First, grab the source for the Nginx module wraps around libmodsecurity:

.. code-block:: console

    [root@server]# git clone https://github.com/SpiderLabs/ModSecurity-nginx.git

Next, grab the Nginx source and verify it:

.. code-block:: console

    [root@server]# wget http://nginx.org/download/nginx-1.10.1.tar.gz

It's also important to grab the developer's signing key and verify the contents
of the download:

.. code-block:: console

    [root@server]# gpg --keyserver pgp.mit.edu --recv a1c052f8
    gpg: requesting key A1C052F8 from hkp server pgp.mit.edu
    gpg: key A1C052F8: public key "Maxim Dounin <mdounin@mdounin.ru>" imported
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   3  signed:   5  trust: 0-, 0q, 0n, 0m, 0f, 3u
    gpg: depth: 1  valid:   5  signed:   0  trust: 4-, 0q, 0n, 0m, 1f, 0u
    gpg: next trustdb check due at 2017-11-22
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)

Next, grab the signature for this tarball:

.. code-block:: console

    [root@server]# wget http://nginx.org/download/nginx-1.10.1.tar.gz.asc

And finally, verify the signature:

.. code-block:: console

    [root@server]# gpg --verify nginx-1.10.1.tar.gz.asc nginx-1.10.1.tar.gz
    gpg: Signature made Tue 31 May 2016 06:58:32 AM PDT using RSA key ID A1C052F8
    gpg: Good signature from "Maxim Dounin <mdounin@mdounin.ru>"
    Primary key fingerprint: B0F4 2533 73F8 F6F5 10D4  2178 520A 9993 A1C0 52F8

From here, configure Nginx with the `--add-module=` option, pointing to the
ModSecurity-nginx module that was previously downloaded:

.. code-block:: console

    [root@server]# tar -zxf nginx-1.10.1.tar.gz
    [root@server]# cd nginx-1.10.1/
    [root@server]# ./configure --add-module=/root/ModSecurity-nginx

From here, simply build and install Nginx:

.. code-block:: console

    [root@server]# make && make install

Configuring libmodsecurity in Nginx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Adding libmodsecurity rules and configuration directives inside Nginx configs
is straightforward. Add the following to the Nginx configuration:

.. code::

    modsecurity on;
    modsecurity_rules '
        SecRuleEngine On
        SecDebugLog /tmp/modsec_debug.log
        SecDebugLogLevel 9
        SecRule ARGS "@streq test" "id:1,phase:1,deny,msg:\'test rule\'"
    ';

These directives can be added inside the `http` block, or one or more `server`
or `location` blocks. Once this is added, reload Nginx. This rule can now be
tested by sending a regular request to Nginx and examining the output:

.. code-block:: console

    [root@server]# curl -D - -s -o /dev/null localhost/
    HTTP/1.1 200 OK
    Server: nginx/1.10.1
    Date: Wed, 13 Jul 2016 18:06:15 GMT
    Content-Type: text/html
    Content-Length: 612
    Last-Modified: Wed, 13 Jul 2016 18:01:34 GMT
    Connection: keep-alive
    ETag: "578681fe-264"
    Accept-Ranges: bytes

The single rule added via the `modsecurity_rules` directive will deny requests
that have the word `test` inside a GET or POST argument. This can be seen by
changing the curl test:

.. code-block:: console

    [root@server]# curl -D - -s -o /dev/null localhost/?a=test
    HTTP/1.1 403 Forbidden
    Server: nginx/1.10.1
    Date: Wed, 13 Jul 2016 18:06:19 GMT
    Content-Type: text/html
    Content-Length: 169
    Connection: keep-alive

A 403 response means that Nginx has blocked the request based on processing the
request with libmodsecurity. From here, libmodsecurity can be customized using
the available directives for ModSecurity (see the
`ModSecurity reference manual <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual>`_
for more information).

Final Notes
~~~~~~~~~~~

It should be noted that libmodsecurity is still in active development, so
certain functionality is subject to change. As with any actively developed
open source project, be sure to check the source code for the most recent
releases.

.. meta::
    :labels: nginx security
