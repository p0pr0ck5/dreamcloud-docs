=======================================
How to use boto-rsync with DreamObjects
=======================================

Overview
~~~~~~~~

`boto-rsync <https://github.com/dreamhost/boto_rsync>`_ is an rsync-like tool
that leverages boto to synchronize files with an object storage service such
as DreamObjects.

   .. note::  The creator of boto-rsync is no longer developing it, so there
              may be bugs and missing features.  DreamHost has forked the code
              (available at the above github URL) and fixed some issues related
              to unicode, multipart uploads, and multiprocessing.

Example Commands
~~~~~~~~~~~~~~~~

   .. note:: Optionally the --delete flag can be included for similar
             functionality to rsync, in order to remove files from the
             destination that don't exist on the source.

**Uploading a file or directory**

.. code-block:: console

    [user@localhost]$ boto-rsync -a ACCESSKEY -s SECRETKEY --endpoint objects-us-west-1.dream.io \
    /SOURCE/PATH s3://DESTINATIONBUCKET/PATH

**Downloading a file or directory**

.. code-block:: console

    [user@localhost]$ boto-rsync -a ACCESSKEY -s SECRETKEY --endpoint objects-us-west-1.dream.io \
    s3://SOURCEBUCKET/PATH /DESTINATION/PATH

.. meta::
    :labels: linux mac boto-rsync rclone
