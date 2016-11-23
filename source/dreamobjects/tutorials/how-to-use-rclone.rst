===================================
How to use rclone with DreamObjects
===================================

Overview
~~~~~~~~

`Rclone <https://rclone.org>`_ is an rsync-like tool for Windows, Mac
OS Linux and other operating systems designed for cloud storage such
as DreamObjects.

It features:

* MD5/SHA1 hashes checked at all times for file integrity
* Timestamps preserved on files
* Partial syncs supported on a whole file basis
* Copy mode to just copy new/changed files
* Sync (one way) mode to make a directory identical
* Check mode to check for file hash equality
* Can sync to and from network, eg two different cloud accounts
* Optional encryption (Crypt)
* Optional FUSE mount (rclone mount)

Configure rclone
~~~~~~~~~~~~~~~~

After you `download <http://rclone.org/downloads/>`_ and install
rclone on your operating system, proceed to configure it for
DreamObjects using the interactive configuration tool:

.. code-block:: console

    [user@localhost]$ $ rclone config
    No remotes found - make a new one
    n) New remote
    s) Set configuration password
    q) Quit config
    n/s/q> n

enter `n` and pick a name you want to assign to this connection, for
example `remote`:

.. code:: bash

    name> remote

Next configure the type of storage as `Amazon S3`, option `2`:

.. code:: bash

    Type of storage to configure.
    Choose a number from below, or type in your own value
     1 / Amazon Drive
       \ "amazon cloud drive"
     2 / Amazon S3 (also Dreamhost, Ceph, Minio)
       \ "s3"
     3 / Backblaze B2
       \ "b2"
     4 / Dropbox
       \ "dropbox"
     5 / Encrypt/Decrypt a remote
       \ "crypt"
     6 / Google Cloud Storage (this is not Google Drive)
       \ "google cloud storage"
     7 / Google Drive
       \ "drive"
     8 / Hubic
       \ "hubic"
     9 / Local Disk
       \ "local"
    10 / Microsoft OneDrive
       \ "onedrive"
    11 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
       \ "swift"
    12 / Yandex Disk
       \ "yandex"
    Storage> 2

Get ready to insert the Access Key and the corresponding Secret Key
you can take from the `DreamObjects Control Panel
<https://panel.dreamhost.com/index.cgi?tree=cloud.objects&>`_:

.. code:: bash

    Get AWS credentials from runtime (environment variables or EC2 meta data if no env vars). Only applies if access_key_id and secret_access_key is blank.
    Choose a number from below, or type in your own value
     1 / Enter AWS credentials in the next step
       \ "false"
     2 / Get AWS credentials from the environment (env vars or IAM)
       \ "true"
    env_auth> 1

    AWS Access Key ID - leave blank for anonymous access or runtime
    credentials.
    access_key_id> your_access_key
    AWS Secret Access Key (password) - leave blank for anonymous access or
    runtime credentials.
    secret_access_key> your_secret_key

Pick option `12` and specify API access point:

.. code:: bash

    Region to connect to.
    Choose a number from below, or type in your own value
       / The default endpoint - a good choice if you are unsure.
     1 | US Region, Northern Virginia or Pacific Northwest.
       | Leave location constraint empty.
       \ "us-east-1"
       / US West (Oregon) Region
     2 | Needs location constraint us-west-2.
       \ "us-west-2"
       / US West (Northern California) Region
     3 | Needs location constraint us-west-1.
       \ "us-west-1"
       / EU (Ireland) Region Region
     4 | Needs location constraint EU or eu-west-1.
       \ "eu-west-1"
       / EU (Frankfurt) Region
     5 | Needs location constraint eu-central-1.
       \ "eu-central-1"
       / Asia Pacific (Singapore) Region
     6 | Needs location constraint ap-southeast-1.
       \ "ap-southeast-1"
       / Asia Pacific (Sydney) Region
     7 | Needs location constraint ap-southeast-2.
       \ "ap-southeast-2"
       / Asia Pacific (Tokyo) Region
     8 | Needs location constraint ap-northeast-1.
       \ "ap-northeast-1"
       / Asia Pacific (Seoul)
     9 | Needs location constraint ap-northeast-2.
       \ "ap-northeast-2"
       / Asia Pacific (Mumbai)
    10 | Needs location constraint ap-south-1.
       \ "ap-south-1"
       / South America (Sao Paulo) Region
    11 | Needs location constraint sa-east-1.
       \ "sa-east-1"
       / If using an S3 clone that only understands v2 signatures
    12 | eg Ceph/Dreamhost
       | set this and make sure you set the endpoint.
       \ "other-v2-signature"
       / If using an S3 clone that understands v4 signatures set this
    13 | and make sure you set the endpoint.
       \ "other-v4-signature"
    region> 12

Next you need to set DreamObject's endpoint API:

.. code:: bash

    Endpoint for S3 API.
    Leave blank if using AWS to use the default endpoint for the region.
    Specify if using an S3 clone such as Ceph.
    endpoint> objects-us-west-1.dream.io

Leave the location constraint empty:

.. code:: bash

    Location constraint - must be set to match the Region. Used when
    creating buckets only.
    Choose a number from below, or type in your own value
     1 / Empty for US Region, Northern Virginia or Pacific Northwest.
       \ ""
     2 / US West (Oregon) Region.
       \ "us-west-2"
     3 / US West (Northern California) Region.
       \ "us-west-1"
     4 / EU (Ireland) Region.
       \ "eu-west-1"
     5 / EU Region.
       \ "EU"
     6 / Asia Pacific (Singapore) Region.
       \ "ap-southeast-1"
     7 / Asia Pacific (Sydney) Region.
       \ "ap-southeast-2"
     8 / Asia Pacific (Tokyo) Region.
       \ "ap-northeast-1"
     9 / Asia Pacific (Seoul)
       \ "ap-northeast-2"
    10 / Asia Pacific (Mumbai)
       \ "ap-south-1"
    11 / South America (Sao Paulo) Region.
       \ "sa-east-1"
    location_constraint> 1

Set the canned ACL based on how you want to use rclone:

.. code:: bash

    Canned ACL used when creating buckets and/or storing objects in S3.
    For more info visit
    http://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl
    Choose a number from below, or type in your own value
     1 / Owner gets FULL_CONTROL. No one else has access rights (default).
       \ "private"
     2 / Owner gets FULL_CONTROL. The AllUsers group gets READ access.
       \ "public-read"
       / Owner gets FULL_CONTROL. The AllUsers group gets READ and WRITE access.
     3 | Granting this on a bucket is generally not recommended.
       \ "public-read-write"
     4 / Owner gets FULL_CONTROL. The AuthenticatedUsers group gets READ access.
       \ "authenticated-read"
       / Object owner gets FULL_CONTROL. Bucket owner gets READ access.
     5 | If you specify this canned ACL when creating a bucket, Amazon S3 ignores it.
       \ "bucket-owner-read"
       / Both the object owner and the bucket owner get FULL_CONTROL over the object.
     6 | If you specify this canned ACL when creating a bucket, Amazon S3 ignores it.
       \ "bucket-owner-full-control"
    acl> private

.. note::

    Read more about `DreamObject's canned ACL <217590537>_`.

Pick `1` for server-side encryption option (DreamObjects doesn't
support it at the moment):

.. code:: bash

    The server-side encryption algorithm used when storing this object in S3.
    Choose a number from below, or type in your own value
     1 / None
       \ ""
     2 / AES256
       \ "AES256"
    server_side_encryption> 1

Finally, review the remote just configured, save it, and exit the
configuration wizard:

.. code:: bash

    Remote config
    --------------------
    [remote]
    env_auth = false
    access_key_id = your_access_key
    secret_access_key = your_secret_key
    region = other-v2-signature
    endpoint = objects-us-west-1.dream.io
    location_constraint =
    acl = private
    server_side_encryption =
    --------------------
    y) Yes this is OK
    e) Edit this remote
    d) Delete this remote
    y/e/d> y

    Current remotes:

    Name                 Type
    ====                 ====
    remote               s3

    e) Edit existing remote
    n) New remote
    d) Delete remote
    s) Set configuration password
    q) Quit config
    e/n/d/s/q> q

Using rclone
~~~~~~~~~~~~

With the remote set, you can list the buckets in it with the command:

.. code:: bash

    [user@localhost]$ rclone lsd remote:
              -1 2016-03-04 02:19:25        -1 samplebucket
              -1 2016-05-16 22:06:53        -1 anotherbucket
              -1 2015-10-15 21:33:25        -1 greatbucket
    2016/09/01 09:35:41
    Transferred:      0 Bytes (0 Bytes/s)
    Errors:                 0
    Checks:                 0
    Transferred:            0
    Elapsed time:       100ms

Make a new bucket:

.. code-block:: console

    [user@localhost]$ rclone mkdir dho:bucket

List the contents of a bucket:

.. code-block:: console

    [user@localhost]$ rclone ls dho:bucket

Sync /home/local/directory to the remote bucket, deleting any excess
files in the bucket:

.. code-block:: console

    [user@localhost]$ rclone sync /home/local/directory remote:bucket

Check `rclone's official documentation <http://rclone.org/docs/>`_ for
more examples on how to use the software.

.. meta::
    :labels: windows linux mac rclone rsync boto-rsync
