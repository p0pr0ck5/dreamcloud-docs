==============================================================
How to Configure LAMP on DreamCompute running Debian or Ubuntu
==============================================================

LAMP (Linux, Apache, MySQL, PHP) stacks are a popular way to create web
service solutions that offer consistent tools and capabilities between
multiple systems.  DreamHost shared, VPS and dedicated hosting services are
based on these same services, and you can use it for your DreamCompute
instance as well.  These instructions assume you run a Debian- or Ubuntu-based
system as they have their own specific configuration and file hierarchy.

Setup Methods
~~~~~~~~~~~~~

There are several ways to setup a LAMP stack on your DreamCompute instance,
from the very manual to the very automated.  We will detail some methods
below.

Automated (Ubuntu only)
-----------------------

The fastest method is to simply tell Ubuntu to install the "task" named
lamp-server.  This is accomplished with apt-get and making sure to use a caret
(^) at the end of the name of the task.

.. code-block:: console

    [user@server]$ sudo apt-get install lamp-server^

The install process asks you to confirm if you wish to install all packages
needed for this task.  Confirm by entering "**y**" and hitting enter.

.. code::

    The following NEW packages will be installed:
        apache2 apache2-bin apache2-data apache2-mpm-prefork libaio1
        libapache2-mod-php5 libapr1 libaprutil1 libaprutil1-dbd-sqlite3
        libaprutil1-ldap libdbd-mysql-perl libdbi-perl libhtml-template-perl
        libmysqlclient18 libterm-readkey-perl mysql-client-5.5 mysql-client-core-5.5
        mysql-common mysql-server mysql-server-5.5 mysql-server-core-5.5 php5-cli
        php5-common php5-json php5-mysql php5-readline ssl-cert
    0 upgraded, 27 newly installed, 0 to remove and 0 not upgraded.
    Need to get 15.2 MB of archives.
    After this operation, 123 MB of additional disk space will be used.
    Do you want to continue? [Y/n]

During this process, the mysql-server installation will ask you to enter and
confirm a root password for MySQL.  It is recommended to provide one.  When
this is complete, please refer to the **Configuration** section of this page
for details on each of these installed packages.

Troubleshooting: If you get a huge amount of 404 File not found errors during
this process, run the following:

.. code-block:: console

    [user@server]$ sudo apt-get update

Then rerun these instructions from the top

Manual Installation
-------------------

To manually install all the needed packages, you can run the following
command:

.. code-block:: console

    [user@server]$ sudo apt-get install apache2 mysql-server php5-cli \
        libapache2-mod-php5 php5-mysqlnd

During this process, the mysql-server installation will ask you to enter and
confirm a root password for MySQL.  It is recommended to provide one.  When
this is complete, please refer to the **Configuration** section of this page
for details on each of these installed packages.

Configuration
~~~~~~~~~~~~~

The below links contain details on the configuration of each part of the newly
installed LAMP stack.

`Apache Directories and Main Configuration Files`_

`MySQL Configuration`_

`PHP Configuration Files`_

.. _Apache Directories and Main Configuration Files: 215879457-How-to-Configure-Apache-on-DreamCompute-Running-Debian-or-Ubuntu

.. _MySQL Configuration: 215879487-How-to-Configure-MYSQL-on-DreamCompute-running-Debian-or-Ubuntu

.. _PHP Configuration Files: 215231198-How-to-Configure-PHP-on-DreamCompute-running-Debian-or-Ubuntu

.. meta::
    :labels: php apache mysql nova debian ubuntu
