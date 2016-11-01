======================================
How to deploy Etherpad on DreamCompute
======================================

Etherpad is a web application that lets you collaborate with others in a text
editor, much like an open source Google Docs alternative.

Setting up a server
~~~~~~~~~~~~~~~~~~~

The first step to deploying Etherpad is to launch a server to run it on. For
example in this tutorial, an Ubuntu Xenial server is used. Read `How to launch
and manage servers with the DreamCompute dashboard
<https://help.dreamhost.com/hc/en-us/articles/215912848-How-to-launch-and-manage-servers-with-the-DreamCompute-dashboard>`__
for information on how to do this.
You also need to expose port 8080 to incoming traffic,
as that is blocked by default. Read `How to configure access and security for
DreamCompute instances
<https://help.dreamhost.com/hc/en-us/articles/215912838-How-to-configure-access-and-security-for-DreamCompute-instances>`__
for information on how to do this

Installing dependencies
~~~~~~~~~~~~~~~~~~~~~~~

.. Note::

    Deploying Etherpad as a service (as done in this article) requires you to
    have root permissions. In order to start a root shell type ``sudo su -``.

Once you have your server up and running the next step is to install all of
Etherpad's dependencies:

.. code-block:: console

    [root@server]# apt-get install gzip git curl python libssl-dev pkg-config \
        build-essential
    [root@server]# apt-get install nodejs npm

Next you must symlink /usr/bin/nodejs to /usr/bin/node because Etherpad will
try to use that path. Most Linux distributions install nodejs in /usr/bin/node.
This step is only necessary on Ubuntu servers since it doesn't install nodejs
in /usr/bin/node because of another package.

.. code-block:: console

    [root@server]# ln -s /usr/bin/nodejs /usr/bin/node

Installing Etherpad
~~~~~~~~~~~~~~~~~~~

Now that all the dependencies are installed the next step is to download
Etherpad and run it. To clone Etherpad using git, run:

.. code-block:: console

    [root@server]# git clone git://github.com/ether/etherpad-lite.git /srv/etherpad-lite

Configuration
-------------

Networking
^^^^^^^^^^

Now comes the configuration of Etherpad. By default it runs on port 9001.
Change it to run on port 8080 by editing /srv/etherpad-lite/settings.json:

.. code-block:: json

    "port" : 9001,

should be changed to:

.. code-block:: json

    "port" : 8080,

.. Note::

    Read `How to configure access and security for DreamCompute instances
    <https://help.dreamhost.com/hc/en-us/articles/215912838-How-to-configure-access-and-security-for-DreamCompute-instances>`__
    for information on how to open port 8080 to traffic

Database
^^^^^^^^

By default Etherpad uses dirtyDB to store its data, but it's recommended you
use something else in a production environment and only use dirtyDB for
testing. This tutorial uses MySQL to store data, but Etherpad supports other
databases such as PostgreSQL and SQLite.

If you don't have MySQL running, follow `this <215879487>`__. Once you have
that running, connect to MySQL and create a database for Etherpad to use:

.. code-block:: console

    [root@server]# mysql -u root -p
    Enter password:
    mysql> CREATE DATABASE etherpad

Finally edit settings.json and delete the configuration for dirtyDB:

.. code-block:: json

    "dbSettings" : {
                   "filename" : "var/dirty.db"
                   },

And add the configuration for MySQL:

.. code-block:: json

    "dbType" : "mysql",
    "dbSettings" : {
                     "user"    : "etherpad",
                     "host"    : "localhost",
                     "password": "ETHERPAD USER PASSWORD",
                     "database": "etherpad",
                     "charset" : "utf8mb4"
                   },

Your configuration may be a bit different depending on how you have MySQL
configured, adjust the values accordingly.

Creating a systemd service
~~~~~~~~~~~~~~~~~~~~~~~~~~

The best way to run Etherpad is to create a systemd service for it and create a
user for it to run as. To create a systemd service copy the following into
/etc/systemd/system/etherpad-lite.service.

.. code::

    [Unit]
    Description=etherpad-lite (real-time collaborative document editing)
    After=syslog.target network.target

    [Service]
    Type=simple
    User=etherpad-lite
    Group=etherpad-lite
    ExecStart=/srv/etherpad-lite/bin/run.sh

    [Install]
    WantedBy=multi-user.target

Next we need to create the user for etherpad-lite to run as.

.. code-block:: console

    [root@server]# adduser --system --home=/srv/etherpad-lite --group etherpad-lite

Now there is an ``etherpad-lite`` user, change the permissions of
/srv/etherpad-lite so that it has access to the directory.

.. code-block:: console

    [root@server]# chown -R etherpad-lite:etherpad-lite /srv/etherpad-lite

Starting Etherpad
~~~~~~~~~~~~~~~~~

Finally start the service and set it to start at boot

.. code-block:: console

    [root@server]# systemctl enable etherpad-lite
    [root@server]# systemctl start etherpad-lite

Etherpad is now running. Confirm it works by going to http://IP:8080. Make
sure to replace "IP" with the IP address of your server.

.. meta::
    :labels: etherpad
