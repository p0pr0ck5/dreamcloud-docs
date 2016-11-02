=====================================================
How to use Python boto scripts to manage DreamObjects
=====================================================

Overview
~~~~~~~~

Boto is a python library that makes it easier to work with services supporting
the Amazon Web Services API, such as DreamObjects. Using boto directly requires
a little bit of Python-scripting knowledge, whereas you can use the same
library via utilities like boto-rsync.

This article describes a few different examples on how you can use boto with
DreamHost. If you're ready to dig into some boto code, the
`boto getting started guide <http://docs.pythonboto.org/en/latest/getting_started.html>`_
is a great place to start.  DreamHost's servers already have boto installed so
you can skip that part in the documentation.

Example Scripts
~~~~~~~~~~~~~~~

Here are some examples of scripts that can be useful for managing your
DreamObjects service.

Checking the size of a bucket
-----------------------------

.. code-block:: python

    #!/usr/bin/python
    # -*- coding: utf-8 -*-
    #Programmatically calculate the total size of your bucket, by iterating over all your objects
    import  boto
    c = boto.connect_s3("ACCESS KEY", "SECRET KEY", host="objects-us-west-1.dream.io")
    b = c.get_bucket("BUCKET NAME")
    s = 0
    for o in b.list():
       s += o.size
    #show you the file size in Megabytes if it's too small to be legible in Gigabytes.
    print str(s * 1.0 / 1024 ** 2) + " MB\n"
    #Shows you the file size in Gigabytes
    print str(s * 1.0 / 1024 ** 3) + " GB"

Purging a directory from a bucket
---------------------------------

    .. note::  Below is a script which removes content from a bucket.  Proceed
               with caution as this will permanently delete your data.

    .. note::  The prefix in the script below can be set to "" to delete all
               data from a bucket, in the case you wish to remove a bucket
               that contains data.

.. code-block:: python

    #!/usr/bin/python
    import boto

    c = boto.connect_s3("ACCESS KEY", "SECRET KEY", host="objects-us-west-1.dream.io")
    b = c.get_bucket("BUCKET NAME", validate=True)

    l = [o for o in b.list(prefix="path/to/directory/to/delete")]
    while len(l):
       s = l[0:1000]
       len(l)
       rs = b.delete_keys(s)
       if len(s) == len(rs.deleted):
           l = l[1000:]

Change all object permissions in a bucket
-----------------------------------------

This script changes all permissions in a bucket to either private (private) or
public (public-read).  Uncomment one of the last two lines for the specific
permission you wish to use.

.. code-block:: python

    #!/usr/bin/python
    import boto

    #Connect to S3
    c = boto.connect_s3("ACCESS KEY", "SECRET KEY", host="objects-us-west-1.dream.io")
    b = c.get_bucket("BUCKET NAME")

    for o in b.list():
    #       o.set_acl('public-read')
    #       o.set_acl('private')

Uploading to a bucket in chunks
-------------------------------

If the file you are attempting to upload is too large, it must be uploaded in
"chunks".  You can use a client that supports multi-part uploads, or use the
boto script below.

    .. note::  This script requires the "FileChunkIO" library.  If you wish to
               use this on a DreamHost server, you will first need to reference
               the Python_ documentation setting up a virtualenv, activating it
               and then running the needed "pip install FileChunkIO" and
               "pip install boto" commands.  If not on a DreamHost server and
               you have root access, using a virtualenv is optional.

.. code-block:: python

    #!/usr/bin/python
    import math, os
    import boto
    from filechunkio import FileChunkIO

    #Connect to S3
    c = boto.connect_s3("ACCESS KEY", "SECRET KEY", host="objects-us-west-1.dream.io")
    b = c.get_bucket("BUCKET NAME")

    #file info
    source_path ='PATH TO YOUR FILE'
    source_size = os.stat(source_path).st_size

    #Create a multipart upload request
    mp = b.initiate_multipart_upload(os.path.basename(source_path))

    #set a chunk size (feel free to change this)
    chunk_size = 100000
    chunk_count = int(math.ceil(source_size / float(chunk_size)))

    #send the file parts using FileChunkIO to create a file-like object
    #that points to a certain byte range within the original file. We set
    #bytes to never exceed the original file size.
    for i in range(chunk_count):
            offset = chunk_size * i
            bytes = min(chunk_size, source_size - offset)
            with FileChunkIO(source_path, 'r', offset=offset,bytes=bytes) as fp:
               mp.upload_part_from_file(fp,part_num=i +1)

    #Finish the upload
    mp.complete_upload()

Additional Examples
~~~~~~~~~~~~~~~~~~~

For more examples of Python code using DreamObjects, please see
`Sample Python Code Using DreamObjects <https://help.dreamhost.com/hc/en-us/articles/214948118-Sample-Python-Code-Using-DreamObjects-S3-compatible-API-And-Boto>`_.

.. _Python: 215489338-Installing-virtualenv-and-custom-modules-in-Python

.. meta::
    :labels: linux mac windows python boto
