==============================================================
How to Configure LAMP on DreamCompute running Fedora or CentOS
==============================================================

LAMP (Linux, Apache, MySQL (we'll use MariaDB instead), PHP) stacks are a
popular way to create web service solutions that offer consistent tools and
capabilities between multiple systems.  DreamHost shared, VPS and dedicated
hosting services are based on these same services, and you can use it for your
DreamCompute instance as well.  These instructions assume you run a Fedora-
(19+) or CentOS-based (7+) system as they have their own specific
configuration and file hierarchy.

Installation
~~~~~~~~~~~~

Install Apache with the following command:

.. code-block:: console

    [root@server]# yum install httpd

Apache can be started and the system told to start it on boot with the
commands:

.. code-block:: console

    [root@server]# systemctl start httpd
    [root@server]# systemctl enable httpd

Install MariaDB client and server with the following command:

.. code-block:: console

    [root@server]# yum install MariaDB-server

MariaDB can be started and the system told to start it on boot with these
commands:

.. code-block:: console

    [root@server]# systemctl start MariaDB
    [root@server]# systemctl enable MariaDB

Both operating systems use "MariaDB" for the service start.

PHP can be installed with the following command:

.. code-block:: console

    [root@server]# yum install php php-mysql

There is no startup script for php.

This concludes the installation portion of this process.

Configuration
~~~~~~~~~~~~~

The below links contain details on the configuration of each part of the newly
installed LAMP stack.

`Apache Directories and Main Configuration Files`_

`MariaDB Configuration`_

`PHP Configuration Files`_

.. _Apache Directories and Main Configuration Files: 217880028-How-to-Configure-Apache-on-DreamCompute-Running-Fedora-or-CentOS

.. _PHP Configuration Files: 215231208-How-to-Configure-PHP-on-DreamCompute-running-Fedora-or-Centos

.. _MariaDB Configuration: 217471877-How-to-Configure-MariaDB-on-DreamCompute-running-Fedora-or-Centos

.. meta::
    :labels: php apache MariaDB mysql fedora centos nova
