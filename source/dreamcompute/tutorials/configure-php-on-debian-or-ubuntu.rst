=============================================================
How to Configure PHP on DreamCompute running Debian or Ubuntu
=============================================================

PHP is a popular server-side scripting language used by many web software and
non-web software packages, and popular with DreamHost customers.  PHP is an
important part of LAMP (Linux, Apache, MySQL, PHP) stacks as well as other AMP
stacks.  You can use it for your DreamCompute instance as well, and the
following information helps you install and configure it.  These instructions
assume you run a Debian- or Ubuntu-based system as they have their own
specific configuration and file hierarchy.

Installing PHP
~~~~~~~~~~~~~~

PHP comes in different configuration packages on Debian and Ubuntu systems to
meet your needs for the command line interface (CLI) client or the apache
module (mod_php aka libphp5.so) or other specializations.

PHP CLI Only
------------

This will install the /usr/bin/php binary and php.ini file.  This is normally
used for running scripts in SSH and for cron jobs, or for development, but
isn't directly usable for websites on apache.

.. code-block:: console

    [root@server]# apt-get install php5-cli

The install process asks you to confirm if you wish to install any additional
packages needed for this installation of PHP.  Confirm by entering "**y**"
and hitting enter:

.. code::

    The following extra packages will be installed:
    libonig2 libqdbm14 php5-common
    Suggested packages:
    php-pear
    The following NEW packages will be installed:
    libonig2 libqdbm14 php5-cli php5-common
    0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.
    Need to get 3,453 kB of archives.
    After this operation, 9,842 kB of additional disk space will be used.
    Do you want to continue [Y/n]?

PHP Apache Module + CLI
-----------------------

This will install the /usr/bin/php binary and php.ini file, as well as the
apache module located at /usr/lib/apache2/modules/libphp5.so and its own
php.ini file.

.. code-block:: console

    [root@server]# apt-get install libapache2-mod-php5

The install process asks you to confirm if you wish to install any additional
packages needed for this installation of PHP.  Confirm by entering "**y**"
and hitting enter:

.. code::

    The following extra packages will be installed:
    php5-cli php5-common php5-json php5-readline
    Suggested packages:
    php-pear php5-user-cache
    The following NEW packages will be installed:
    libapache2-mod-php5 php5-cli php5-common php5-json php5-readline
    0 upgraded, 5 newly installed, 0 to remove and 5 not upgraded.
    Need to get 4,852 kB of archives.
    After this operation, 20.4 MB of additional disk space will be used.
    Do you want to continue? [Y/n]

The install process will enable the php5 module for you, and restart apache.
If you wish to double check that it is enabled, you can use the a2enmod
program:

.. code-block:: console

    [root@server]# a2enmod php5
    Module php5 already enabled

If you do disable or enable modules, make sure to restart apache with the
service command:

.. code-block:: console

    [root@server]# service apache2 restart
    [ ok ] Restarting web server: apache2 ... waiting .

PHP Modules for Databases
-------------------------

There are additional packages available to install modules for various
database engines.  Here are some of the more common ones that can use apt-get
to be installed.

* **php5-mysqlnd**

  This package contains the mysql, mysqli and PDO mysql modules for PHP using
  the MySQL Native Driver which is newer and generally faster.  There is a
  standard "php5-mysql" package available that contains all these same modules
  using the older non-Native Driver is you need to connect to older MySQL
  servers.

* **php5-pgsql**

  The package containing the pgsql and pdo_pgsql modules for use with
  PostgreSQL.

* **php5-sqlite**

  The package containing the sqlite3 and pdo_sqlite modules for working with
  SQLite.

Optional PHP Modules
--------------------

These modules add additional features of functionality to PHP depending on
your needs.  Some of the more common ones will be listed below, with a link
to all available packages at the bottom.

* **php5-curl**

  The cURL module for PHP to allow for downloading from HTTP, FTP and other
  sources.

* **php5-ffmpeg**

  The ffmpeg libraries and php5-gd will be installed with this package, to
  allow retrieving meta-data from video and images, as well as some video
  frame editing.

* **php5-gd**

  The GD library provides image manipulation features which can be useful for
  some applications.

* **php5-imagick**

  The ImageMagick library is wrapped into PHP when this package is installed,
  allowing for some image manipulation features.

Additional modules for php5 can be found on the respective packages pages for
each operating system.

`Debian Packages for PHP5
<https://packages.debian.org/search?keywords=php5-&searchon=names&suite=stable&section=all>`_

`Ubuntu Packages for PHP5
<http://packages.ubuntu.com/search?keywords=php5-&searchon=names&suite=trusty&section=all>`_

Configuration Files
~~~~~~~~~~~~~~~~~~~

There are two configuration files for PHP that are used for the CLI and the
apache modules specifically.  The thinking is that you would want saner
defaults for PHP when accessed from the web because it is probably your
production environment.  The CLI on the other hand, is more likely your
development environment and should be less restricted.

* **/etc/php5/cli/php.ini**

  The primary configuration file for PHP when executed from the CLI.  It has
  no memory_limit at all, and allows all functions (including `POSIX PHP
  functions <http://php.net/manual/en/ref.pcntl.php>`_).

* **/etc/php5/apache2/php.ini**

  The primary configuration file for PHP when ran from the apache module.  It
  has a memory_limit of 128MB, and disallows all functions related to `POSIX
  PHP functions <http://php.net/manual/en/ref.pcntl.php>`_ for security and
  safety reasons.

.. meta::
    :labels: php ubuntu debian
