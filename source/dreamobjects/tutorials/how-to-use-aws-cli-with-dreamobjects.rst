====================================
How to use AWS CLI with DreamObjects
====================================

Overview
~~~~~~~~

`AWS CLI <https://aws.amazon.com/cli/>`_ is:
    * a command-line tool for uploading, retrieving, and managing data in
      Amazon S3 and other Cloud Storage Service Providers that use the S3
      protocol such as DreamHost DreamObjects.
    * best suited for power users who don't fear command line.
    * ideal for scripts, automated backups triggered from cron, and so on.

The following instructions help you install and configure AWS CLI to work with
DreamObjects.

Installing AWS CLI
~~~~~~~~~~~~~~~~~~

Depending on the operating system you are running, there are several options.

For Windows there are files in .msi format that can be installed directly.

For Mac and Linux, the suggested installation is through python "pip".  To do
this, run the following command on your local computer:

.. code-block:: console

    [user@localhost]$ pip install awscli

To install the client on a DreamHost shared, VPS or dedicated server will
require the use of python virtualenv and then using its "pip" to install the
AWS CLI client locally with the same pip install command above.  Please see
the Python_ article for
specifics on how to accomplish this.

Configuring AWS CLI
~~~~~~~~~~~~~~~~~~~

This client has features that apply to many services offered by Amazon, but
for this tutorial we only are concerned with using the S3 functionality in
combination with DreamObjects.  To use the S3 features, you must ensure a few
things:

1. Newer versions of AWS CLI may have a signature_version variable set that is
   incompatible with DreamObjects.  Setting the value to empty if one exists
   will ensure compatibility.  You can check and change this value with these
   commands.

.. code-block:: console

    [user@localhost]$ aws configure get default.s3.signature_version
    s3v4
    [user@localhost]$ aws configure set default.s3.signature_version ""

2. Run the following command to input your access and secret keys for AWS CLI
   to store them encrypted for you.  Accept the default region and output
   format by hitting enter.

.. code-block:: console

    [user@localhost]$ aws configure
    AWS Access Key ID [None]:
    AWS Secret Access Key [None]:
    Default region name [None]:
    Default output format [None]:

Example Commands
~~~~~~~~~~~~~~~~

**Making a bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 mb s3://newbucketname
    make_bucket: s3://newbucketname/

**Listing all buckets**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 ls
    2016-01-27 20:14:46 newbucketname

**Uploading a file into a bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 cp testfile.txt s3://newbucketname/testfile.txt
    upload: ./testfile.txt to s3://newbucketname/testfile.txt

**Listing the contents of a bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 ls s3://newbucketname
    2016-01-27 19:30:21       8803 testfile.txt

**Downloading a file from a bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 cp s3://newbucketname/testfile.txt testfile.txt
    download: s3://newbucketname/testfile.txt to ./testfile.txt

**Deleting a file in a bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 rm s3://newbucketname/testfile.txt
    delete: s3://newbucketname/testfile.txt

**Deleting an empty bucket**

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 rb s3://newbucketname/
    remove_bucket: s3://newbucketname/

**Sync a directory and its files to or from a bucket**

This will only upload new and changed files, and not delete any files.  You
can specify other params such as --delete to remove files from the destination
that aren't on the source.  An additional useful flag is --acl which accepts
values such as "private" or "public-read".

.. code-block:: console

    [user@localhost]$ aws --endpoint-url https://objects-us-west-1.dream.io s3 sync syncdir s3://newbucketname/
    upload: syncdir/file3 to s3://newbucketname/file3
    upload: syncdir/file1 to s3://newbucketname/file1
    upload: syncdir/file2 to s3://newbucketname/file2

.. _Python: 215489338-Installing-virtualenv-and-custom-modules-in-Python

.. meta::
    :labels: linux mac windows aws awscli
