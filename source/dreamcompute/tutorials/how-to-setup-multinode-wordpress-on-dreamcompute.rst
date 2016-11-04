=================================================
How to deploy multinode WordPress on DreamCompute
=================================================

.. container:: table_of_content

    - :ref:`Architecture`
    - :ref:`Requirements`
    - :ref:`MySQL`
    - :ref:`NFS`
    - :ref:`WordPress`
    - :ref:`HAProxy`

Setting up a multi-node infrastructure allows you to increase scalability
and durability of a WordPress installation. Scalability means that as
your site grows, becomes more popular, and acquires more users, you
can easily add more servers to your site to spread the load caused by
the increased web traffic; durability means it's harder for your site
to break and become unavailable.

The following tutorial describes how to setup a multi-node WordPress site.

.. _Architecture:

Architecture
~~~~~~~~~~~~

This example website is set up with four servers running on a private
network. The first server runs HAProxy to spread requests evenly
between two webservers, and the last server runs MySQL and NFS. MySQL
provides the database backend for WordPress, and NFS volume is used to
share the same WordPress site and content between the webservers, so
they serve identical websites.

.. code::

                                          +-----------+
    app1.domain.io           +----------->|    Web1   |---+    10.10.10.5
        |  +-------------+   | 10.10.10.3 +-----------+   |   +-----------+
        +->|   HAProxy   |---+                            +-->| MySQL/NFS |
           +-------------+   | 10.10.10.4 +-----------+   |   +-----------+
                             +----------->|    Web2   |---+
                                          +-----------+

.. _Requirements:

Requirements
~~~~~~~~~~~~

The requirements for a DreamCompute multinode WordPress site are:

* 4 Ubuntu servers (Ubuntu 16.04 is recommended as it is newer and will
  receive support for a longer period)
* Private networking is enabled in your DreamCompute account
* 1 floating IP address

To enable private networking in your account, read
`What is DreamCompute Predictable Bill
<217744568-What-is-DreamCompute-Predictable-Bill>`_.

.. _MySQL:

Initial setup
~~~~~~~~~~~~~

The first step is to create four servers on your private network:

* 1 for a load balancer
* 2 for webservers
* 1 for MySQL and NFS

Associate a floating IP address with your load balancer, which is where the
traffic from the public comes from. This is also your jump host for
logging into your servers that are not exposed to the public.

You can read `How to log in to a server that doesn't have a public IP address
<215879497>`_ for more information.

Setting up a MySQL server
~~~~~~~~~~~~~~~~~~~~~~~~~

Installation
------------

The following example illustrates how to build a website from the back forward,
which is why MySQL comes first. Follow this tutorial on `How to
deploy MySQL on an Ubuntu server <215879487>`_.

Configuration
-------------

There are a few configuration changes you must make to your MySQL server
after it is installed. Edit the configuration file,
/etc/mysql/mysql.conf.d/mysqld.conf on Ubuntu 16.04, and change the bind
address from 127.0.0.1, localhost, to the private IP address of your MySQL
server:

.. code::

    bind-address            = IP

Next, you must create a database for WordPress and a user that has
access to that database. Connect to the database as root by running
the following:

.. code-block:: console

    [user@mysqlserver]$ mysql -u root -p
    Enter password:
    mysql>

Then, to create the database and a user that has access to it, run the
following:

.. code-block:: sql

    mysql> CREATE DATABASE wordpress;
    mysql> GRANT ALL ON wordpress.* TO wordpress@'10.10.10.%' IDENTIFIED by 'PASSWORD';


The first command creates the database named "wordpress" — you can name
the database whatever you want, just remember what you called it. The
second command grants complete control to the "wordpress" database to
a user named "wordpress" coming from an IP address within the 10.10.10.0
to 10.10.10.255 range. This means that only someone on your private
network can access the database. This is very secure as it means your
database can't be brute forced from someone on the internet unless
they manage to access your private network.

Security groups
---------------

The final step to setting up your database server is to add a security
group to your server that allows TCP connections on port 3306 from an IP
address inside your private network. To do this, follow the tutorial on
`How use to the DreamCompute dashboard to add security groups to a server <215912838>`_.
You must open the TCP port 3306 to the CIDR of your private network, which
is 10.10.10.0/24 in this example:

.. _NFS:

Setting up a NFS shared volume
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Network File System (NFS) is a way to mount a part of a filesystem from one
server to another, over a network. NFS is necessary because your site
will have two webservers, and their webroots must be synchronized. NFS
is one of the easiest ways to do this.

Installation
------------

You can install NFS on a separate server, but also you can simply use the
same server as your MySQL server.

To install NFS packages on Ubuntu:

.. code-block:: console

    [user@mysqlserver] sudo apt update
    [user@mysqlserver] sudo apt install nfs-kernel-server

Configuration
-------------

To configure NFS, create a directory to be exported to other servers
and make its content owned by the www-data user and group:

.. code-block:: console

    [user@mysqlserver] sudo mkdir -p /exports/www
    [user@mysqlserver] sudo chown www-data:www-data /exports/www

.. note:: By default, PHP processes on Ubuntu run as www-data. If you
          have modified the default PHP user, you should adapt the chown
          command above.

Then, configure NFS to export that directory. Edit /etc/exports and add:

.. code::

    /exports/www  10.10.10.0/24(rw,sync,no_subtree_check)

where "/exports/www" is the directory to export and "10.10.10.0/24" is the range of IP
addresses to allow to mount this directory.

.. Note::

    You may need to restart the NFS service for your configuration
    changes to take effect. You can do this with ``sudo service
    nfs-kernel-server restart``.

Security groups
---------------

The final step to complete your NFS server is to add a security
group to the server allowing TCP connections on ports 111 and 2049
from any IP address inside your private network. To do this, follow
the `tutorial <215912838>`_ on using the DreamCompute dashboard to add security
groups to a server.

.. _WordPress:

Setting up WordPress
~~~~~~~~~~~~~~~~~~~~

You must set up two webservers and install the LAMP stack on
each, but keep in mind that WordPress' PHP code is only added
once, as the code is installed to a volume shared by both servers.

Installation
------------

Installing a LAMP stack
^^^^^^^^^^^^^^^^^^^^^^^

WordPress requires a webserver stack, such as Apache2 and PHP
interpreter:

.. code-block:: console

    [user@web] sudo apt update
    [user@web] sudo apt install apache2
    [user@web] sudo apt install php-curl php-gd php-mbstring php-mcrypt \
        php-xml php-xmlrpc php-common libapache2-mod-php php-cli php-mysql

Mounting /exports/www
^^^^^^^^^^^^^^^^^^^^^

Before installing WordPress, configure both webservers to mount the
/exports/www/ directory from the NFS server.

.. code-block:: console

    [user@web] sudo apt install nfs-common
    [user@web] sudo mkdir -p /var/www
    [user@web] sudo mount NFS-SERVER-IP:/exports/www /var/www

Then, edit the /etc/fstab file on both webservers to automatically mount
/exports/www from the NFS server at boot, and add the following:

.. code::

    NFS-SERVER-IP:/exports/www /var/www nfs defaults 0 0

Substitute "NFS-SERVER-IP" with the actual IP address of your NFS server.

Installing WordPress
^^^^^^^^^^^^^^^^^^^^

Follow the `Step-by-step guide to deploy WordPress on DreamCompute
<220973627>`_ as you normally would on a single-node computer, but
skip the sections about installing the LAMP stack and setting up the
database as those steps are already complete.

.. Note::

    Since the `wp core install` requires an extra flag when you run your database on
    another server, use the flag `--dbhost=MYSQL-SERVER-IP` to specify the IP
    address of the MySQL server.

Configuration
-------------

In /var/www (the root of your WordPress site), edit the file
config.php and add the following to it:

.. code-block:: php

    if ($_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https') $_SERVER['HTTPS']='on';

You must add this code as the website is served over SSL and WordPress must
be configured to do that.

.. Note::

    If you change the ownership of the files in your /var/www directory or if
    you need to edit the files as root, you must do it from your NFS
    server, because the NFS server does not allow a client to perform root
    actions to files that it serves.

.. _HAProxy:

Setting up HAProxy loadbalancer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

HAProxy is software that balances HTTP requests between multiple
webservers, distributing the workload across them.

Setting up a loadbalancer
-------------------------

The only setup needed for HAProxy is to create an Ubuntu 16.04 server on your
private network and assign a floating IP to it.

Installation
------------

HAProxy is packaged in Ubuntu:

.. code-block:: console

    [user@haproxy] sudo apt update
    [user@haproxy] sudo apt install haproxy

Configuration
-------------

Getting an SSL cert using Let's Encrypt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's Encrypt is a service that makes it easier to get SSL certificates and
secure your website.

First, download the Let's Encrypt tools:

.. code-block:: console

    [user@haproxy] sudo -s
    [root@haproxy] cd /opt
    [root@haproxy] git clone git://github.com/letsencrypt/letsencrypt

Then, request a certificate:

.. code-block:: console

    [root@haproxy] cd /opt/letsencrypt
    [root@haproxy] ./letsencrypt-auto certonly --standalone -d example.com

When you finish answering the questions, you should have new SSL
certificates and keys.

Finally, put the certs into the right place:

.. code-block:: console

    [root@haproxy] mkdir -p /etc/ssl/example.com/privkey.pem
    [root@haproxy] cd /etc/letsencrypt/live/example.com/
    [root@haproxy] cat fullchain.pem privkey.pem > /etc/ssl/example.com/privkey.pem
    [root@haproxy] chmod 600 /etc/ssl/example.com/privkey
    [root@haproxy] chmod 700 /etc/ssl/example.com

Configuring HAProxy
^^^^^^^^^^^^^^^^^^^

To configure HAProxy, change /etc/haproxy/haproxy.cfg to the
following:

.. code::

    global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    maxconn 4096
    user haproxy
    group haproxy
    daemon

    defaults
    log    global
    mode    http
    option    httplog
    option    dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

    option forwardfor
    option http-server-close
    stats enable
    stats auth admin:stats
    stats uri /haproxyStats

    frontend https-in
        bind HAPROXY-IP:443 ssl crt /etc/ssl/example.com/privkey.pem
        reqadd X-Forwarded-Proto:\ https
        default_backend blog_cluster

    frontend http-in
        bind HAPROXY-IP:80
        reqadd X-Forwarded-Proto:\ http
        redirect scheme https code 301 if !{ ssl_fc }
        default_backend blog_cluster

    # Define hosts
    acl blog_host hdr(host) -i example.com

    # Figure out which one to use
    use_backend blog_cluster if blog_host

    backend blog_cluster
    balance leastconn
    option httpclose
    server node1 WEB1-IP:80 cookie A check
    server node2 WEB2-IP:80 cookie A check

Substitute HAPROXY-IP, WEB1-IP, and WEB2-IP with the IP addresses of your
servers. The HAPROXY-IP is the private IP of your HAProxy server and not the
floating IP you assigned it.

This HAProxy configuration causes HAProxy to listen on ports 80 and 443. It
also redirects HTTP requests to HTTPS, which is ideal so that visitors to
your site are redirected to a secure connection when they try to make an
insecure request over HTTP.

If everything works correctly, you should be able to point your
browser to the floating IP of your HAProxy server and see the
graphic interface to finish the WordPress installation.

.. meta::
    :labels: wordpress nfs mysql haproxy
