======================================================
How to Migrate Instances Between DreamCompute Clusters
======================================================

Introduction
~~~~~~~~~~~~

DreamCompute offers multiple clusters (also often called availability zones)
which are independent OpenStack installations with their own servers, storage
and control panel.  Some clusters have different features, such as SSD storage
or different hardware that is useful for a given task.  Migrating instances
and data between clusters is not automated at this time, and this guide will
show you how to accomplish this yourself.

This guide assumes that you are comfortable working with SSH, and some
command line utilities such as `dd <http://man7.org/linux/man-pages/man1/dd.1.html>`_
and `glance <http://docs.openstack.org/developer/python-glanceclient/man/glance.html>`_.

Things To Keep in Mind
~~~~~~~~~~~~~~~~~~~~~~

Here are a few things to keep in mind and plan while doing a migration.

* **IP Addresses Will Change**

  Each cluster has assigned blocks of IP addresses, and therefore floating IPs
  or public IPv4 and IPv6 addresses cannot be transferred between clusters.  If
  you are using private networks (required in US-East 1 and optional in
  US-East 2, the specific assigned 10.x.x.x address may also change.

* **SSH Keys**

  Each cluster manages its SSH Keys separately, so if you have your keys
  already setup in US-East 1, you will have to setup the same keys or new
  ones in US-East 2.  If OpenStack generated the SSH Key for you, it let you
  download the private key but the public key is what you would need for an
  import.  You could grab it from ~/.ssh/authorized_keys on an
  instance that used the key.  For the instances themselves, the
  authorized_keys file isn't overwritten, only appended, and so whatever keys
  are currently setup will continue to work after the move.

* **Plan A Maintenance Window**

  It is safest to move the data when the instance is not running, to avoid open
  files causing corruption or other odd behavior.  The copying of the data is
  generally pretty quick for smaller volumes, but for larger volumes could take
  some time.  The copy needs to complete before service can be restored.  Also,
  DNS will need to be updated from the previous public IPs to the new ones.
  Depending on the TTL (time to live) of your DNS provider, this process can be
  a matter of minutes, or take 24 hours.  DreamHost managed DNS can have the
  TTL changed from 4 hours to 5 minutes if you contact support, and can help
  minimize propagation time.

* **Ephemeral Instances**

  Ephemeral instances cannot have snapshots taken, and when shutdown cannot
  have their data accessed by another instance.  These instances can only be
  migrated while running, so it is best to shut down as many services like
  MySQL, apache, and so on to limit possible corruption.  Please see the last
  section below on how to migrate running instances.

Migrate a Volume-backed Instance using Glance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For this type of move, we will have to delete the instance so that it leaves
behind the volume to migrate.  This method requires that you didn't checkmark
the "delete on terminate" checkbox when you created your instance.  If you did,
please skip to the last section on migrating running instances.  If you
continue you may permanently destroy and lose your data.

As an overview, we are going to setup the following to accomplish this task.

.. code::

        SOURCE CLUSTER         DESTINATION CLUSTER

       +---------------+     +----------------------+     +-------------------+
       | Temp Instance |---->| Glance Image Service |---->| Migrated Instance |
       +---------------+     +----------------------+     +-------------------+
               |(mount)
       +----------------+
       | Volume To Copy |
       +----------------+

Here we go!

1.  Create a new instance, using the smallest available flavor, to be used as
a copy machine.  For this guide, I will be using Ubuntu 14.04, however the
commands should be similar on any Ubuntu system.  I would recommend making the
instance ephemeral, since we won't be planning to keep it.

2.  Install the needed software to work with glance on this new instance.

.. code-block:: console

    [root@server]# apt-get install python-dev python-pip
    [root@server]# pip install python-openstackclient
    [root@server]# pip install python-glanceclient

After this, run "glance help" and check for any other modules that it says are
missing.  Install them with:

.. code-block:: console

    [root@server]# pip install MODULENAME

3.  Setup your OpenStack RC file for the DESTINATION cluster on this new
instance, which can be downloaded from its Access & Security -> API Access menu
in the dashboard.  Either upload the file to your instance, or copy/paste its
contents into a file on this instance.  Once you are run, you can run it like
so.

.. code-block:: console

    [root@server]# vi dreamcompute-CLUSTER.sh
    <paste the contents, save>
    [root@server]# . dreamcompute-CLUSTER.sh

It will then prompt you to "Please enter your OpenStack Password:", and go
ahead and do that.

If you run a command like the below, it should output the current OS images
in the destination cluster.

.. code-block:: console

    [root@server]# glance image-list

4.  Delete the instance that you wish to move, freeing up its volume to be
attached to the above newly created instance.

5.  Attach the volume to the new instance, in the Volumes menu by clicking the
drop-down on the right side, and then "Edit Attachments".

6.  On the new instance, check "dmesg" for the drive letter, or you can check
the usual names for it, until you find the volume.

.. code-block:: console

    [root@server]# fdisk -l /dev/vdb | grep Disk
    [root@server]# fdisk -l /dev/vdc | grep Disk

One of those should match the size of the volume you are trying to move.  Make
note of the drive letter (the /dev/vdX part).

7.  Now we will copy the data to glance, using dd and piping it directly.
Don't forget to change the drive letter in the example to the one you found
above, and change any text in all CAPS to suit your taste.

.. code-block:: console

    [root@server]# dd if=/dev/vdX | glance --os-image-api-version 2 \
        image-create \ --name "INSTANCENAME" --is-public false --disk-format \
        raw --container-format bare

8.  Wait while this runs, and if successful it should output the info about the
new image that was created.

9.  You are now ready to go to the DESTINATION cluster to start up a new
instance and to select the image we just uploaded.  It is best to use a volume
instead of ephemeral in this situation if the data is meant to be persistent.

Migrate an Ephemeral Instance using Glance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This type of migration is not recommended.  It may be necessary in some
situations however and so is included here.

1.  Shut down as many services as possible, such as database servers, http
servers, etc, leaving hopefully just default system tools and sshd running.

2.  Install the needed software to work with glance on this new instance.

.. code-block:: console

    [root@server]# apt-get install python-dev python-pip
    [root@server]# pip install python-openstackclient
    [root@server]# pip install python-glanceclient

After this, run "glance help" and check for any other modules that it says are
missing.  Install them with:

.. code-block:: console

    [root@server]# pip install MODULENAME

3.  Setup your OpenStack RC file for the DESTINATION cluster on this new
instance, which can be downloaded from its Access & Security -> API Access menu
in the dashboard.  Either upload the file to your instance, or copy/paste its
contents into a file on this instance.  Once you are run, you can run it like
so.

.. code-block:: console

    [root@server]# vi dreamcompute-CLUSTER.sh
    <paste the contents, save>
    [root@server]# . dreamcompute-CLUSTER.sh

It will then prompt you to "Please enter your OpenStack Password:", and go
ahead and do that.

If you run a command like the below, it should output the current OS images
in the destination cluster.

.. code-block:: console

    [root@server]# glance image-list

4.  Determine the drive letter by examining the output of "df -h" for the root
(/) filesystem.  Generally this will be /dev/vda1.

5.  Now we will copy the data to glance, using dd and piping it directly.
Change any text in all CAPS to suit your taste.

.. code-block:: console

    [root@server]# dd if=/dev/vda | glance --os-image-api-version 1 image-create \
        --name "INSTANCENAME" --is-public false --disk-format raw \
        --container-format bare

6.  Wait while this runs, and if successful it should output the info about the
new image that was created.

7.  You are now ready to go to the DESTINATION cluster to start up a new
instance and to select the image we just uploaded.  It is best to use a volume
instead of ephemeral in this situation if the data is meant to be persistent.

.. meta::
    :labels: glance migrate image
