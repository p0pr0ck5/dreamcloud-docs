================================================================
How to Configure Apache on DreamCompute Running Fedora or CentOS
================================================================

Apache is the most widely used HTTP server on the internet, and DreamHost uses
it extensively as the default HTTP server for all hosting products.

.. note::

    These instructions assume you run a Fedora- or CentOS-based system as
    they have their own specific configuration and file hierarchy.

Installing Apache
~~~~~~~~~~~~~~~~~

To install Apache on your system, run the following command:

.. code-block:: console

    [root@server]# yum install httpd

* The install process asks you to confirm if you wish to install any
  dependency packages needed for Apache.
* Enter "**y**" and hit enter to confirm.

In order to start Apache run

.. code-block:: console

    [root@server]# service httpd start

This may display an error about the lack of a configuration, but it
will start anyways.

.. code::

    Starting httpd: httpd: apr_sockaddr_info_get() failed for centos65
    httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
    [  OK  ]

You likely want apache to start on boot, and this can be configured
with:

.. code-block:: console

    [root@server]# chkconfig httpd on

If you visit the public IP in your browser for your DreamCompute
instance, you are able to see the 'getting started' page.  You can
find this IP on the Instances (IP Address column) or
Access & Security (floating ips tab) panel pages.

*The default page displays the following when Apache successfully
installs:*

.. code::

    Apache 2 Test Page powered by CentOS
    or
    Fedora Test Page

    This page is used to test the proper operation of the Apache HTTP
    server after it has been installed. If you can read this page it
    means that the Apache HTTP server installed at this site is
    working properly.

Apache Directories and Main Configuration Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The /etc/httpd2 directory
-------------------------

This directory contains all the configuration files for your Apache
server, and symlinks to other parts of the Apache install such as the
logs and modules directories.

.. code-block:: console

    [root@server]# ls /etc/httpd
    conf  conf.d  conf.modules.d  logs  modules  run

conf
----

This directory by default only contains the main Apache config file
named "httpd.conf", and the "magic" file used for determining MIME
types.  The httpd.conf file is the only user-editable file and is well
documented what each part of it does.  For making additions to this
file, you can edit it directly to add your changes but it is
recommended to create new .conf files in the conf.d directory instead
for ease of management.

conf.d
------

All files with the .conf extension in this directory will be loaded
last by httpd.conf and in alphabetical order.  The default Apache
install from Fedora and CentOS have different basic contents in these
directories, but one common file is welcome.conf to load the default
Apache startup page when nothing else is configured.  Additional files
will be added here by the administrator to configure Apache for the
sites and features needed.

conf.modules.d (Fedora only)
----------------------------

This directory contains configuration files only used for loading
modules and their options.  It is recommended to make any non-standard
modifications for modules in the conf.d directory instead.  CentOS
does not have this directory, relying on the httpd.conf file or conf.d
entries for changes to modules instead.

Virtual Hosts
~~~~~~~~~~~~~

Virtual hosts define each site so that Apache knows what it should do
when it receives a request.  The Apache configuration process on
Fedora and CentOS is less defined compared to the Debian/Ubuntu
setups, which can allow for more flexibility.  For easier management
it is recommended to create individual .conf files to configure
specific services in the /etc/httpd/conf.d directory, however you are
free to create the files and their contents any way you desire.

/etc/httpd/conf.d/welcome.conf
------------------------------

The welcome.conf file defines what Apache should do when it gets a
request that matches no other virtual hosts.  If you only expect to
have one site on your DreamCompute instance, you could use this file
and no others if you prefer.  For those with multiple sites, this can
be used to instruct the visitor that they may have done something
wrong, or redirect them to another site.

/etc/httpd/conf.d/YOURSITEHERE.conf
-----------------------------------

* For each site you wish to configure, we recommended you name a file
  similar to your site name in the **/etc/httpd/conf.d/** directory.
* There are several example virtual hosts available on the Apache Wiki
  `Example Vhosts page <http://wiki.apache.org/httpd/ExampleVhosts>`_
  but you can view a basic one listening on port 80 (http) with custom
  logging here:

.. code-block:: apacheconf

    <VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/www.example.com

    CustomLog /var/log/httpd/www.example.com-access.log combined
    ErrorLog /var/log/httpd/www.example.com-error.log
    </VirtualHost>

Alternatively, if you wish to specify the ip instead of "**\***" you
can use the following command replacing 1.1.1.1 with your real ip
address:

.. code-block:: apacheconf

    <VirtualHost 1.1.1.1:80>

Managing virtual host files
-----------------------------

If you create a .conf file for each site and wish to enable or disable
that site, all this would require is removing or moving that sites
specific .conf file out of the /etc/httpd/conf.d directory and then
reloading Apache. Alternatively, you could comment out the entire file
by adding "#" to the front of each line.  You can reload Apache via
the command:

.. code:: bash

    [root@server]# service httpd reload

.. meta::
    :labels: apache fedora centos linux
