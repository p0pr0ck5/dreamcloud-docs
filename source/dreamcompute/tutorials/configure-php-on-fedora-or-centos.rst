=============================================================
How to Configure PHP on DreamCompute running Fedora or CentOS
=============================================================

PHP is a popular server-side scripting language used by many web software and
non-web software packages, and popular with DreamHost customers.  PHP is an
important part of LAMP (Linux, Apache, MySQL, PHP) stacks as well as other AMP
stacks.  You can use it for your DreamCompute instance as well, and the
following information helps you install and configure it.  These instructions
assume you run a Fedora- or CentOS-based system as they have their own
specific configuration and file hierarchy.

Installing PHP
~~~~~~~~~~~~~~

PHP comes in different configuration packages on Debian and Ubuntu systems to
meet your needs for the command line interface (CLI) client or the apache
module (mod_php aka libphp5.so) or other specializations.

PHP Apache Module + CLI Only
----------------------------

To install the main PHP package, run the following yum command:

.. code-block:: console

    [root@server]# yum install php

This installs the php-cli, php apache module and apache if it isn't selected
already.

PHP Modules for Databases
-------------------------

There are additional packages available to install modules for various
database engines.  Here are some of the more common ones that can use apt-get
to be installed.

* **php-mysqlnd**

  This package contains the mysql, mysqli and PDO mysql modules for PHP using
  the MySQL Native Driver which is newer and generally faster.

* **php-pgsql**

  The package containing the pgsql and pdo_pgsql modules for use with
  PostgreSQL.

Optional PHP Modules
--------------------

These modules add additional features of functionality to PHP depending on
your needs.  Some of the more common ones will be listed below, with a link
to all available packages at the bottom.

* **php-gd**

  The GD library provides image manipulation features which can be useful for
  some applications.

Additional modules for php5 can be found on the respective packages pages for
each operating system or by issuing the below yum command:

.. code-block:: console

    [root@server]# yum list php-\*

`Fedora Packages for PHP5 <https://admin.fedoraproject.org/pkgdb/packages/php-%2A/>`_

Configuration Files
~~~~~~~~~~~~~~~~~~~

For Fedora- or CentOS-based systems, the default is to have only one
configuration file for all methods of using php.

* **/etc/php.ini**

  The php.ini file used for the PHP CLI, apache module and any other php
  methods.

.. meta::
    :labels: php fedora centos
