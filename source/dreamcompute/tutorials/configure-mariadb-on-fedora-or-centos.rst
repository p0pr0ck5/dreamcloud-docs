=================================================================
How to Configure MariaDB on DreamCompute running Fedora or CentOS
=================================================================

Installing MariaDB
~~~~~~~~~~~~~~~~~~

To install MariaDB on your system, run the following commands with the desired
MariaDB version, such as:

.. code-block:: console

    [root@server]# yum install mariadb-server

The install process asks you to confirm if you wish to install any additional
packages needed for this installation of MariaDB.  Confirm by entering "**y**"
and hitting enter.

.. code::

    Dependencies Resolved

    ==============================================================================================
    Package                          Arch           Version                    Repository    Size
    ==============================================================================================
    Installing:
     mariadb-server                  x86_64         1:5.5.44-2.el7.centos      base          11 M
    Installing for dependencies:
     libaio                          x86_64         0.3.109                    base          24 k
     mariadb                         x86_64         1:5.5.44-2.el7.centos      base         8.9 M
     perl-Compress-Raw-Bzip2         x86_64         2.061-3.el7                base          32 k
     perl-Compress-Raw-Zlib          x86_64         1:2.061-4.el7              base          57 k
     perl-DBD-MySQL                  x86_64         4.023-5.el7                base         140 k
     perl-DBI                        x86_64         1.627-4.el7                base         802 k
     perl-Data-Dumper                x86_64         2.145-3.el7                base          47 k
     perl-IO-Compress                noarch         2.061-2.el7                base         260 k
     perl-Net-Daemon                 noarch         0.48-5.el7                 base          51 k
     perl-PlRPC                      noarch         0.2020-14.el7              base          36 k

    Transaction Summary
    ==============================================================================================
    Install  1 Package (+10 Dependent packages)

    Total download size: 21 M
    Installed size: 108 M
    Is this ok [y/d/N]:

After the installation completes, you should start the service and configure
mariadb by running the following commands as root

.. code-block:: console

    [root@server]# systemctl start mariadb
    [root@server]# mysql_secure_installation

If you enter a password, another dialog box will come up for you to re-enter
the password to confirm.

If you want MariaDB to start automatically after a reboot, run the following as
root

.. code-block:: console

    [root@server]# systemctl enable mariadb

Configuring and Using MariaDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The configuration files are stored in the /etc and /etc/my.cnf.d directories.
If any changes are made, you must restart MariaDB for it to read the them.
This can be done by logging in as root and running:

.. code-block:: console

    [root@server]# systemctl restart mariadb

Main configuration file /etc/my.cnf
-----------------------------------

This is the main configuration file for MariaDB.  There are a few settings you
may wish to change:

* **bind**
    The ip address that MariaDB is listening to.  It can only listen to one ip
    address at any time.  By default it will listen to 127.0.0.1 (aka
    localhost), meaning that the MariaDB service will only be accessible from
    the instance it is installed on.  If you want to connect to it from other
    DreamCompute instances, you can change this to your instances IPv4 or IPv6
    IP address.  Here is what an IPv6 configured MariaDB bind variable looks
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

    [root@server]# /usr/bin/mysqladmin -u root password 'new-password' -p

Connecting to MariaDB with a shell
----------------------------------

To connect to your new MariaDB install, to setup new databases or configure new
users, you can run these commands.

Via socket (should work even if "bind" is changed in my.cnf):

.. code-block:: console

    [user@server]$ mysql -S /var/lib/mysql/mysql.sock -u root -p

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
    :labels: mariadb mysql fedora centos
