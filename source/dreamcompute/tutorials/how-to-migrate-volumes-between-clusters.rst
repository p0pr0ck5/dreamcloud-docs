====================================================
How to Migrate Volumes Between DreamCompute Clusters
====================================================

Introduction
~~~~~~~~~~~~

DreamCompute offers multiple clusters (also often called availability zones)
which are independent OpenStack installations with their own servers, storage
and control panel.  Some clusters have different features, such as SSD storage
that is useful for a given data storage plan.  Migrating data between clusters
is not automated at this time, and this guide will show you how to accomplish
this yourself.

This guide assumes that you are comfortable working with SSH, and some
command line utilities such as `dd <http://man7.org/linux/man-pages/man1/dd.1.html>`_.

Things To Keep in Mind
~~~~~~~~~~~~~~~~~~~~~~

Here are a few things to keep in mind and plan while doing a migration.

* **Volumes Only**

  Please see the article on migrating instances if you wish to move instances.
  This guide focuses solely on migrating volumes, which is useful if you have
  multiple volumes attached to an instance, and after migrating the instance
  you want to then migrate the remaining volumes.

  Also, this method does not work with ephemeral storage and is intended only
  for volume to volume copying.

* **Plan A Maintenance Window**

  It is safest to move the data when the volume has no running services or open
  files, to avoid corruption or other odd behavior.  The copying of the data is
  generally pretty quick for smaller volumes, but for larger volumes could take
  some time.  The copy needs to complete before service can be restored.

Migrate a Volume using SSH and dd
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For this type of move, we will assume you have stopped all services using the
data on the volume, safely unmounted it on the instance using it, and detached
it from the instance in the Volumes menu in the dashboard.

As an overview, we are going to setup the following to accomplish this task.

.. code::

        SOURCE CLUSTER       DESTINATION CLUSTER

       +---------------+     +---------------+
       | Temp Instance |---->| Temp Instance |
       +---------------+     +---------------+
               |(mount)              |(mount)
      +----------------+      +--------------+
      | Volume To Copy |      |  New Volume  |
      +----------------+      +--------------+

Here we go!

1.  Create two new instances, using the smallest available flavor, to be used
as copy machines, one each in the source and destination clusters.  For this
guide, I will be using Ubuntu 14.04, however the commands should be similar on
any Ubuntu system.  I would recommend making the instance ephemeral, since we
won't be planning to keep it.

2.  For simplicity, we will use passwords for the two temp instances to connect
instead of SSH keys, however you can do it either way you prefer.  To setup
the instances for password authentication, turn it on with the below
commands.

.. code-block:: console

    <login to each instance>
    [user@server]$ sudo su -
    [root@server]# sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    [root@server]# sed -i -e 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
    [root@server]# service ssh restart
    [root@server]# passwd root
    <set a password>

3.  Create a matching volume in the destination cluster, that is the same size
or larger than the source volume.

4.  Attach the source volume to the instance in the source cluster, to the
temporary instance.  Attach the destination volume to the instance in the
destination cluster.  There is no need to mount them.

5.  Determine the drive letter of the volumes on both instances.  Generally
/dev/vda will be the boot drive of your instance, so it will be /dev/vdb or
/dev/vdc.  You can check for it with a couple commands:

.. code-block:: console

    [root@server]# fdisk -l /dev/vdb | grep Disk
    [root@server]# fdisk -l /dev/vdc | grep Disk

The one that matches the size of the volume is the one to use.  They may have
different drive letters on each instance, so take note of that.

6.  Now we can copy the data using dd and ssh.  For this we will login to the
instance on the destination cluster, and use the IPv6 address for simplicity.
Replace IPV6-OF-SOURCE-INSTANCE with the IPv6 address of the source instance
and the first /dev/vdX with the drive letter of the source volume, and the
second /dev/vdX with the drive letter of the destination volume.

.. code-block:: console

    [root@destserver]# ssh root@IPV6-OF-SOURCE-INSTANCE "dd if=/dev/vdX | gzip -1 -" | gunzip - | dd of=/dev/vdX

7.  Detach the destination volume from the instance, and check that it has the
data you want by trying to boot it or attach it to another instance.  If all
looks correct, you can destroy both temporary instances and you are done.

.. meta::
    :labels: migrate volume
