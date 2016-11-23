=================================
How to Use duck with DreamObjects
=================================

Overview
~~~~~~~~

.. figure:: images/cyberduck.png

`Duck <https://duck.sh/>`_ is a command-line interface tool used for file
transfers. duck provides a command-line interface (CLI) to connect via FTP,
SFTP, or webDAV and can also connect to cloud storage.

This article describes how to use duck to connect to your DreamObjects
buckets.

How to install duck
~~~~~~~~~~~~~~~~~~~

duck is ready to use on DreamHost hosting machines so no installation is
necessary to use it.

If you'd like to use duck on your Mac, Windows, or Linux machine, follow the
instructions on the duck `installation
<https://trac.cyberduck.io/wiki/help/en/howto/cli#Installation>`_ page.

Downloading the Connection Profile
----------------------------------

Again, this step is not needed if you're using duck from a DreamHost hosting
machine.

Similar to Cyberduck, the duck CLI tool uses a connection profile as a
pre-configured settings file to connect to DreamObjects.

#. Download the `DreamObjects Connection Profile
   <https://objects-us-west-1.dream.io/applications/DreamObjects-CLI.cyberduckprofile>`_
   if you're going to run duck on your computer.
#. Once you've downloaded the DreamObjects Connection Profile, move it into
   the proper directory:

    **Linux and Windows-**

    .. code::

        ~/.duck/profiles

    **Mac-**

    .. code::

        ~/Library/Application Support/Cyberduck/Profiles

This enables you to natively connect to your buckets using Duck.

Connecting to your DreamObjects bucket
--------------------------------------

Please refer to duck's CLI `usage
<https://trac.cyberduck.io/wiki/help/en/howto/cli#Usage>`_ page.

Alternatively, type the following when using duck for usage information:

.. code-block:: console

    [user@localhost]$ duck --help

Helpful commands
----------------

    **provides the switches and syntax to perform duck commands**

    .. code-block:: console

        [user@localhost]$ duck --help

    **lists objects in your bucket**

    .. code-block:: console

        [user@localhost]$ duck --list dreamhost://mybucketname/ -i (your access key) -p \
        (your secret key)

    **lists objects in a bucket folder**

    .. code-block:: console

        [user@localhost]$ duck --list dreamhost://mybucketname/mysubfolder -i (your access key) \
        -p (your secret key)

    **uploads file to the bucket, mybucket/**

    .. code-block:: console

        [user@localhost]$ duck --username myuser --upload dreamhost://mybucket/ \
        my.file.to.upload

    *When prompted, enter your Access key then your Secret key.*

    **opens a file to edit**

    .. code-block:: console

        [user@localhost]$ duck --edit dreamhost://mybucketname/yourfile -i (your access key) \
        -p (your secret key)

    *Be sure to set the permission to the bucket or file to public or you will
    get a forbidden error.*

.. meta::
    :labels: duck linux mac windows backup
