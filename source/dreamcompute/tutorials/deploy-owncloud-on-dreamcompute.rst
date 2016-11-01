=====================================================
Step-by-step guide to deploy ownCloud on DreamCompute
=====================================================

Preparation
~~~~~~~~~~~

In this tutorial we are going to install ownCloud on two DreamCompute
instances, one for the application itself and one for the database it uses.
We'll install and configure all necessary components without making use of
automatic configuration management systems. Future tutorials will cover
automation.

First you need to deploy 2 Ubuntu 14.04LTS virtual machines. It's better to
boot volume backed instances as they are permanent as opposed to ephemeral
disks. You can do this in the `web UI <215912848>`_ or with the `nova client
<215912778>`_.  Once you have those instances up and running, you need to add
a security group to the instance that runs the database so that it allows TCP
on port 3306, the MySQL/MariaDB port. That can be done with the `web
interface <215912838>`_ or the `nova command line client <216511637>`_ as well.

Installing MariaDB
~~~~~~~~~~~~~~~~~~

In order to install MariaDB on your database server, first login to the server
with:

.. code-block:: console

    [user@localhost]$ ssh user@$IP

changing the IP to your server's public IP address. Then run

.. code-block:: console

    [user@localhost]$ sudo su -
    [root@server]#

this creates a root shell, which you will need because you have to have
administrator rights to install things system-wide. Now that you have a root
shell you can install mariadb by running:

.. code-block:: console

    [root@server]# apt-get install mariadb-server

It will ask for a root password for the Database, enter whatever you want and
remember it, you will need it later.

Configuring MariaDB
~~~~~~~~~~~~~~~~~~~

Changing the bind address
-------------------------

Open the /etc/mysql/my.conf file in an editor and edit the line that says

.. code::

    bind-address            = 127.0.0.1

and change it to

.. code::

    bind-address            = $IP

where **$IP** is the IP address of the DB server.

.. note::

    If you have private networking enabled, this will be the private IP address
    and not the floating IP if your DB server has one.

This makes the database listen to connections from it's IP address instead of
only listening on 127.0.0.1, which is localhost.

Allowing root login from a foreign IP address
---------------------------------------------

Now our database will listen to connections from other servers, but we have
to allow root to login from another IP address. We do this by logging into the
DB as root with

.. code-block:: console

    [root@server]# mysql -u root -p

then enter the root password for your database. Then run:

.. code-block:: sql

    use mysql;
    update user set host='$IP' where user='root' and host='$HOSTNAME';
    flush privileges;

where **$IP** is the IP address of your application instance, and **$HOSTNAME**
is the hostname of your database server.

.. note::

    If you want to allow root login from any IP address, change $IP to '%', but
    this is not recommended, especially if your database server has a public IP
    address, because then anyone can try access it.

now restart the mariadb service so the new configs are loaded by running:

.. code-block:: console

    [root@server]# service mysql restart

Installing the ownCloud application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Installing Dependencies
-----------------------

Now that we have a database that ownCloud can use, we need to deploy the
frontend application. First login to the server that you will be
installing ownCloud on. Create a root shell again by running

.. code-block:: console

    [user@localhost]$ sudo su -
    [root@server]#

Then run

.. code-block:: console

    [root@server]# apt-get install apache2 libapache2-mod-php5
    [root@server]# apt-get install php5-gd php5-json php5-mysql php5-curl
    [root@server]# apt-get install php5-intl php5-mcrypt php5-imagick

to install the packages that ownCloud requires to run.

Downloading ownCloud
--------------------

Now we need to download the actual ownCloud application. Do this by going to
https://owncloud.org/install/#instructions-server in a browser and right click
the *.tar.bz2* link and click *copy link location* then in your root shell run

.. code-block:: console

    [root@server]# wget $URL

where **$URL** is the URL you just copied. This will download a compressed
copy of the ownCloud application. Decompress the file by running

.. code-block:: console

    [root@server]# bzip2 -d owncloud-9.0.0.tar.bz2
    [root@server]# tar -xvf owncloud-9.0.0.tar

owncloud-9.0.0.tar.bz2 is the name of the file you just downloaded and
owncloud-9.0.0.tar is the directory created by running the bzip2 command. The
version numbers for your download might be different from mine.
This should create a directory called "owncloud" in your current directory.

Setting up the owncloud directory
---------------------------------

First we need to copy ownCloud to the right directory. We will be running it
out of /var/www/owncloud. To copy it run

.. code-block:: console

    [root@server]# cp -R owncloud /var/www/

Now we want to change the permissions of the owncloud directory so that the web
user, www-data in our case, can access it. Do this by running

.. code-block:: console

    [root@server]# chown -R www-data:www-data /var/www/owncloud

Configuring Apache
------------------

Now that we have ownCloud in the right place, we need to configure Apache to
use it. To do this we must create a file in /etc/apache2/sites-available called
"owncloud.conf" and make it's contents

.. code-block:: apacheconf

    Alias /owncloud "/var/www/owncloud/"

    <Directory /var/www/owncloud/>
      Options +FollowSymlinks
      AllowOverride All

     <IfModule mod_dav.c>
      Dav off
     </IfModule>

     SetEnv HOME /var/www/owncloud
     SetEnv HTTP_HOME /var/www/owncloud

    </Directory>

Then symlink /etc/apache2/sites-enabled/owncloud.conf to
/etc/apache2/sites-available/owncloud.conf by running

.. code-block:: console

    [root@server]# ln -s /etc/apache2/sites-available/owncloud.conf \
        /etc/apache2/sites-enabled/owncloud.conf

ownCloud also needs certain apache modules to run properly, enable them by
running

.. code-block:: console

    [root@server]# a2enmod rewrite

You should also use SSL with ownCloud to protect login information and data,
Apache installed on Ubuntu comes with a self-signed cert. To enable SSL using
that cert run

.. code-block:: console

    [root@server]# a2enmod ssl
    [root@server]# a2ensite default-ssl
    [root@server]# service apache2 restart

Finishing the Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~

Now everything is configured on the server, open a browser and visit
https://IP/owncloud where **IP** is the IP address of your application instance.
Create an admin account using the web interface. Then fill in the details for
the database. The database user is "root", the password is the root password
for the database, the host is the IP address of your database
server, and the database name can be set to anything, I recommend "owncloud".
Then continue and **BAM** you have a working ownCloud.

.. meta::
    :labels: owncloud
