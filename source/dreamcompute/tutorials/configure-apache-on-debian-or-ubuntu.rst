================================================================
How to Configure Apache on DreamCompute Running Debian or Ubuntu
================================================================

Apache is the most widely used HTTP server on the internet, and DreamHost uses
it extensively as the default HTTP server for all hosting products.

.. note::

    These instructions assume you run a Debian- or Ubuntu-based system
    as they have their own specific configuration and file hierarchy.

Installing Apache
~~~~~~~~~~~~~~~~~

To install Apache on your system, run the following commands:

.. code-block:: console

    [user@server]$ sudo apt-get update
    [user@server]$ sudo apt-get install apache2

* The install process asks you to confirm if you wish to install any
  additional packages needed for this instance of Apache.
* Enter "**y**" and hit enter to confirm.

.. code-block:: console

    [user@server]$ sudo apt-get install apache2
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    The following extra packages will be installed:
    apache2-mpm-worker apache2-utils apache2.2-bin apache2.2-common
    libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
    ssl-cert
    Suggested packages:
    apache2-doc apache2-suexec apache2-suexec-custom openssl-blacklist
    The following NEW packages will be installed:
    apache2 apache2-mpm-worker apache2-utils apache2.2-bin
    apache2.2-common libapr1 libaprutil1 libaprutil1-dbd-sqlite3
    libaprutil1-ldap ssl-cert
    0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
    Need to get 1,487 kB of archives.
    After this operation, 5,673 kB of additional disk space will be used.
    Do you want to continue [Y/n]? y

* When it completes, the apache HTTP server is installed and runs with
  its default configuration.
* If you visit the public IP in your browser for your DreamCompute
  instance, you are able to see the 'getting started' page.
* You can find this IP on the Instances (IP Address column) or
  Access & Security (floating ips tab) panel pages.

*The default page displays the following when apache successfully
installs:*

.. code::

    It works!
    This is the default web page for this server.
    The web server software is running but no content has been added, yet.

Apache Directories and Main Configuration Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The /etc/apache2 Directory
--------------------------

This directory contains all the configuration files for your Apache
server. The top of most configuration files in this directory include
details on its general purpose.

The basic functionality of the files are described in the next
section.

/etc/apache2/apache2.conf File
------------------------------

This is the main configuration file that ultimately controls how
Apache functions.  While it is possible to completely configure your
sites and modules directly in this file, instead it is recommended to
use smaller individual files for each of your sites and modules for
simplicity.  This is made possible by the "**Include**" directive to
insert other files into the apache2.conf at runtime.  Some of the
values of interest in here are:

* **Timeout**
    Length of time in seconds that Apache attempts to fulfill a
    request. Default:  300

* **KeepAlive**
    Define if persistent connections is allowed, which allows more
    than one request per connection.
    Default:  On

* **MaxKeepAliveRequests**
    Define the maximum number of requests allowed for each KeepAlive
    persistent connection. Default:  100

* **KeepAliveTimeout**
    Define the number of seconds to wait for another request before
    ending the KeepAlive persistent connection.  Default:  5

* **MPM Configuration**
    Debian and Ubuntu have different Apache packages that are
    optimized for different situations.  Each package is a different
    flavor of MPM (multi-processing module) and settings for each are
    defined near the end of this file.  The packages available are
    apache2-mpm-prefork, apache2-mpm-worker, and apache2-mpm-event,
    with apache2-mpm-worker being the default.  This is the default
    threaded implementation of Apache and is recommended for
    high-traffic sites due to its speed and smaller memory footprint.

To check which MPM configuration (and modules) exist, run the
following:

.. code-block:: console

    [user@server]$ sudo apache2 -l

*The enabled modules are listed, ending with .c . In this case, the
worker module is enabled:*

.. code::

    Compiled in modules:
    core.c
    mod_log_config.c
    mod_logio.c
    mod_version.c
    worker.c
    http_core.c
    mod_so.c

Virtual Hosts
~~~~~~~~~~~~~

Virtual hosts define each site so that Apache knows what it should do
when it receives a request.

/etc/apache2/sites-available/default
------------------------------------

* Defines what Apache should do when it gets a request that matches no
  other virtual hosts.
* If you only expect to have one site on your DreamCompute instance,
  you could use this file and no others if you prefer.
* For those with multiple sites, this can be used to instruct the
  visitor that they may have done something wrong, or redirect them to
  another site.

/etc/apache2/sites-available/YOURSITEHERE
-----------------------------------------

* For each site you wish to configure, it is recommended you name a file
  similar to your site name in the **/etc/apache2/sites-available/**
  directory.
* There are several example virtual hosts available on Apache's wiki
  `Example Vhosts page <http://wiki.apache.org/httpd/ExampleVhosts>`_
  but you can view a basic one for listening on port 80 (http) with
  custom logging here:

.. code-block:: apacheconf

    <VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/www.example.com

    CustomLog /var/log/apache/www.example.com-access.log combined
    ErrorLog /var/log/apache/www.example.com-error.log
    </VirtualHost>

* Alternatively, if you wish to specify the ip instead of "**\***" you
  can use the following command replacing 1.1.1.1 with your real ip
  address:

.. code-block:: apacheconf

    <VirtualHost 1.1.1.1:80>

Managing Virtual Host files
---------------------------

When you have your sites virtual host file setup, you can
enable/disable it by entering the following commands:

**sudo a2ensite**
    Provides a list of sites files that you can enable.

**sudo a2dissite**
    Provides a list of sites files you can disable.

**service apache2 reload**
    Reloads apache to make the change live after you enable or disable a site.

These commands create a symlink for your sites file from
/etc/apache2/sites-enabled to its corresponding file in
/etc/apache2/sites-available.

Modules
~~~~~~~

Modules can be enabled or disabled by the following commands:

.. code-block:: console

    [user@server]$ sudo a2enmod

    [user@server]$ sudo a2dismod

* When you run the command, it displays a list of modules available to
  enable or disable.
* After you enable or disable a site, reload Apache to make the change
  live by using the following command:

    .. code-block:: console

        [user@server]$ service apache2 reload

.. meta::
    :labels: apache ubuntu debian linux
