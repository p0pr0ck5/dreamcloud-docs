===============================================================
How to Configure MYSQL on DreamCompute running Debian or Ubuntu
===============================================================

MySQL is the worlds second most popular database engine (after SQLite) and a
very common choice for a open-source relational database management system
(RDBMS).  MySQL is an important part of LAMP (Linux, Apache, MySQL, PHP)
stacks as well as other AMP stacks, and is supported by many web software
packages.  MySQL is the only relational database software offered by DreamHost
for its shared, VPS and dedicated hosting products, except for SQLite.  You
can use it for your DreamCompute instance as well, and the following
information helps you install and configure it.  These instructions assume you
run a Debian- or Ubuntu-based system as they have their own specific
configuration and file hierarchy.

Installing MySQL
~~~~~~~~~~~~~~~~

Debian and Ubuntu sometimes offer multiple versions of MySQL in case your
needs require a later version, however it is usually best to go with their
defaults if no specific version is required.

Debian 7.0 wheezy:

* mysql-server / mysql-server-5.5 - default and only version available

Ubuntu 14.04 LTS:

* mysql-server / mysql-server-5.5 - default version of MySQL 5.5
* mysql-server-5.6 - alternative version of MySQL 5.6

To install MySQL on your system, run the following commands with the desired
mysql version, such as:

.. code-block:: console

    [user@server]$ sudo apt-get update
    [user@server]$ sudo apt-get install mysql-server

The install process asks you to confirm if you wish to install any additional
packages needed for this installation of MySQL.  Confirm by entering "**y**"
and hitting enter.

.. code-block:: console

    [user@server]$ sudo apt-get install mysql-server
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    The following extra packages will be installed:
    libaio1 libdbd-mysql-perl libdbi-perl libhtml-template-perl
    libmysqlclient18 mysql-client-5.5 mysql-common mysql-server-5.5
    mysql-server-core-5.5
    Suggested packages:
    libipc-sharedcache-perl libterm-readkey-perl tinyca
    The following NEW packages will be installed:
    libaio1 libdbd-mysql-perl libdbi-perl libhtml-template-perl
    libmysqlclient18 mysql-client-5.5 mysql-common mysql-server
    mysql-server-5.5
    mysql-server-core-5.5
    0 upgraded, 10 newly installed, 0 to remove and 1 not upgraded.
    Need to get 9,696 kB of archives.
    After this operation, 97.0 MB of additional disk space will be used.
    Do you want to continue [Y/n]?

Before the installation completes, a dialog will pop up asking you to specify
a MySQL server root password.  The root user is the default admin user and
should have a strong password.

.. code::

    Configuring mysql-server-5.5
    While not mandatory, it is highly recommended that you set a
    password for the MySQL administrative "root" user.
    If this field is left blank, the password will not be changed.
    New password for the MySQL "root" user:

If you enter a password, another dialog box will come up for you to re-enter
the password to confirm.

When it completes, the MySQL server is installed and runs with its default
configuration.

Configuring and Using MySQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The configuration files are stored in the /etc/mysql directory.  If any
changes are made, you must restart MySQL for it to read the them.  This can be
done by running:

.. code-block:: console

    [user@server]$ sudo service mysql restart

Main configuration file /etc/mysql/my.cnf
-----------------------------------------

This is the main configuration file for MySQL.  There are a few settings you
may wish to change:

* **bind**
    The ip address that MySQL is listening to.  It can only listen to one ip
    address at any time.  By default it will listen to 127.0.0.1 (aka
    localhost), meaning that the MySQL service will only be accessible from
    the instance it is installed on.  If you want to connect to it from other
    DreamCompute instances, you can change this to your instances IPv4 or IPv6
    IP address.  Here is what an IPv6 configured mysql bind variable looks
    like:

.. code::

    bind-address            = 2607:f298:6050:8a28:f816:3eff:fe62:c9c3

* **max_allowed_packet**
    The largest size allowed for a single packet, which normally is only
    relevant for restoring backups.  If a backup was created on a server with
    a high setting for this value, it may have difficulty restoring on another
    machine with a lower setting for this value.  The default is 16M.

Resetting the root password
---------------------------

If you forget the root password, it can be reset by running this command and
entering in a new password twice.

.. code-block:: console

    [user@server]$ dpkg-reconfigure mysql-server-5.5

Connecting to mysql shell
-------------------------

To connect to your new mysql install, to setup new databases or configure new
users, you can run these commands.

Via socket (should work even if "bind" is changed in my.cnf):

.. code-block:: console

    [user@server]$ mysql -S /var/run/mysqld/mysqld.sock -u root -p

Via IP address:

.. code-block:: console

    [user@server]$ mysql -h 10.10.10.X -u root -p

Via localhost:

.. code-block:: console

    [user@server]$ mysql -h localhost -u root -p

or just:

.. code-block:: console

    [user@server]$ mysql -u root -p

.. meta::
    :labels: mysql debian ubuntu
