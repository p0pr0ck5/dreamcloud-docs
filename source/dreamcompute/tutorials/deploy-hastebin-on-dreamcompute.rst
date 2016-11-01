======================================
How to deploy Hastebin on DreamCompute
======================================

Hastebin is a simple self-hosted pastebin alternative.

Setting up a server
~~~~~~~~~~~~~~~~~~~

The first step to deploying Hastebin is to launch a server to run it on. For
example in this tutorial, an Ubuntu Xenial server is used. Read `How to launch
and manage servers with the DreamCompute dashboard
<https://help.dreamhost.com/hc/en-us/articles/215912848-How-to-launch-and-manage-servers-with-the-DreamCompute-dashboard>`__
for information on how to do this.
You also need to expose port 8080 to incoming traffic,
as that is blocked by default. Read `How to configure access and security for
DreamCompute instances
<https://help.dreamhost.com/hc/en-us/articles/215912838-How-to-configure-access-and-security-for-DreamCompute-instances>`__
for information on how to do this.

Installing dependencies
~~~~~~~~~~~~~~~~~~~~~~~

.. Note::

    Deploying Hastebin as a service (as done in this article) requires you to
    have root permissions. In order to start a root shell type ``sudo su -``.

Once you have your server up and running the next step is to install all of
Hastebin's dependencies:

.. code:: console

    [root@server]# apt-get update
    [root@server]# apt-get install nodejs npm

Next you must symlink /usr/bin/nodejs to /usr/bin/node because Hastebin will
try to use that path. Most Linux distributions install nodejs in /usr/bin/node.
This step is only necessary on Ubuntu servers since it doesn't install nodejs
in /usr/bin/node because of another package.

.. code:: console

    [root@server]# ln -s /usr/bin/nodejs /usr/bin/node

Installing Hastebin
~~~~~~~~~~~~~~~~~~~

Now that all the dependencies are installed the next step is to download
Hastebin and run it. To clone Hastebin using git, run:

.. code:: console

    [root@server]# git clone https://github.com/seejohnrun/haste-server.git /srv/haste-server

Then to install the application, run:

.. code:: console

    [root@server]# cd /srv/haste-server
    [root@server]# npm install

Configuration
-------------

Networking
^^^^^^^^^^

Now comes the configuration of Hastebin. By default it runs on port 7777.
Change it to run on port 8080 by editing /srv/haste-server/config.js:

.. code:: console

    "port" : 7777,

should be changed to:

.. code:: console

    "port" : 8080,

.. Note::

    Read `How to configure access and security for DreamCompute instances
    <https://help.dreamhost.com/hc/en-us/articles/215912838-How-to-configure-access-and-security-for-DreamCompute-instances>`__
    for information on how to open port 8080 to traffic.

Database
^^^^^^^^

By default Hastebin uses redis to store its data, we're going to use a simpler
solution, flat files.
Edit config.js and delete the configuration for redis:

.. code-block:: javascript

    "storage": {
      "type": "redis",
      "host": "0.0.0.0",
      "port": 6379,
      "db": 2,
      "expire": 2592000
    },

and replace it with the configuration for using flat files:

.. code-block:: javascript

    "storage": {
      "type": "file",
      "path": "./data"
    },

.. Note::

    Flat files will not scale as well as a database will, so if you expect to
    have lots of users, be sure to use a database for your data.

Creating a systemd service
~~~~~~~~~~~~~~~~~~~~~~~~~~

The best way to run Hastebin is to create a systemd service for it and create a
user for it to run as. To create a systemd service copy the following into
/etc/systemd/system/haste-server.service.

.. code::

    [Unit]
    Description=haste-server (online code snippet sharing tool)
    After=syslog.target network.target

    [Service]
    WorkingDirectory=/srv/haste-server
    Type=simple
    User=haste-server
    Group=haste-server
    ExecStart=/usr/bin/node server.js

    [Install]
    WantedBy=multi-user.target

Next create the user for haste-server to run as.

.. code:: console

    [root@server]# adduser --system --home=/srv/haste-server --group haste-server

Now there is an 'haste-server' user, change the permissions of
/srv/haste-server so that it has access to the directory.

.. code:: console

    [root@server]# chown -R haste-server:haste-server /srv/haste-server

Starting Hastebin
~~~~~~~~~~~~~~~~~

Finally start the service and set it to start at boot:

.. code:: console

    [root@server]# systemctl enable haste-server
    [root@server]# systemctl start haste-server

Hastebin is now running. Confirm it works by going to http://IP:8080. Make
sure to replace "IP" with the IP address of your server.

.. meta::
    :labels: Hastebin
