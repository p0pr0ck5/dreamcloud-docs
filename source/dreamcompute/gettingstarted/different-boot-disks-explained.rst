==============================================================
What's the difference between ephemeral and volume boot disks?
==============================================================

Different Boot Sources
----------------------

There are several different kinds of sources to boot from in the DreamCompute
panel, but they all need to create some sort of virtual disk for the virtual
machine to boot. The virtual disk can be either an *ephemeral* or *volume* boot
disk. In the DreamCompute panel you see the options:

* **Image**: Creates an ephemeral disk from the image you choose.
* **Volume**: Boots an instance from an existing bootable volume.
* **Image (create a new volume)**: Creates a bootable volume from the image
  you choose and then boots an instance from it.
* **Volume snapshot (create a new volume)**: Creates a volume from the
  volume snapshot you choose and boots an instance from it.

Ephemeral Boot Disks
--------------------

Ephemeral disks are virtual disks that are created for the sole purpose of
booting a virtual machine and should be thought of as temporary.

Ephemeral disks are useful if you aren't worried about needing to duplicate an
instance or destroy an instance and save the data. You can still mount a volume
on an instance that is booted from an ephemeral disk and put any data that
needs to be saved on it, instead of using the volume as the root of your OS.

- **Do not use up volume quota**: If you have more instance quota, you can
  always boot it from an ephemeral disk even if you don't have any volume
  quota left.
- **Are destroyed when the instance is terminated**: This means if you want to
  temporarily delete an instance to free up some instance quota, you can't
  without losing your data.

Volume Boot Disks
-----------------

Volumes are a more permanent form of storage than ephemeral disks and can be
used to boot from as well as a mountable block device.

Volume boot disks are useful if you need an easy way to duplicate instances and
back them up with snapshots, or if you need a more reliable storage solution
for your instance than an ephemeral disk. If you use them, you should plan
ahead so that you have enough quota for all of the instances you want to boot.

- **Can be snapshotted**: Useful for duplicating instances or having a copy of
  an instance at a certain point in time

- **Does not get destroyed when you terminate the instance (Unless you
  check the "Delete on Terminate" box)**: You can terminate the instance and
  your data will still exist as a volume that you can boot from later

- **Uses your volume quota**: This can be pricey if you have lots of instances,
  or take lots of snapshots

.. meta::
    :labels: boot volume
