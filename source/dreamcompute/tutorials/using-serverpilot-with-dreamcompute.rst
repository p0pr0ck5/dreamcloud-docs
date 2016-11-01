==============================================================
Registering a DreamCompute server with ServerPilot using Shade
==============================================================

This article assumes you have

* An account on ServerPilot and your `client ID and API key
  <https://serverpilot.io/community/articles/how-to-use-the-serverpilot-api.html>`__
  for that account.
* An `openrc.sh file
  <228047207-How-to-download-your-DreamCompute-openrc-file>`__

Authentication
~~~~~~~~~~~~~~

Start by setting the environment variables that tell Shade how to authenticate
to Dreamcompute, do this by running the following and typing in your password
when it asks for it:

.. code-block:: console

    [user@localhost]$ source openrc.sh

Getting a server ID and server API key from ServerPilot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first step in this process is to request a new server ID and server API
key from ServerPilot. You also set the name of the server you want to create
here.

.. literalinclude:: examples/serverpilot.py
    :language: python
    :start-after: step-1
    :end-before: step-2

Next you make the request to the ServerPilot API to create new a new server ID
and API key.

.. literalinclude:: examples/serverpilot.py
    :language: python
    :start-after: step-2
    :end-before: step-3

Now you have a server ID and server API key stored in the ``response_json``
dictionary that you can use with the server you want to register with
ServerPilot.

Launching a server and registering it with ServerPilot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pass the ServerPilot installer as a task to be executed as soon as the
new server is created, using cloud-init:

.. literalinclude:: examples/serverpilot.py
    :language: python
    :start-after: step-3
    :end-before: step-4

Then set variables for the image, flavor, and key pair to launch the server
with.

.. literalinclude:: examples/serverpilot.py
    :language: python
    :start-after: step-4
    :end-before: step-5

Change the ``key_name`` file to be the name of your key pair on DreamCompute so
that you can SSH into the server. The ``image`` and ``flavor_id``
variables can also be modified to deploy a different image or a different size
server.

Finally, connect to DreamCompute with Shade and request the building of the
server. For more information about Shade, read our documentation on `how
to use Shade with DreamCompute <214836997>`__.

.. literalinclude:: examples/serverpilot.py
    :language: python
    :start-after: step-5

Once the script runs and finishes, go to `ServerPilot.io
<https://manage.serverpilot.io/#servers>`__, click on servers
and you should see your new server (it may take up to a couple minutes for the
installation script to finish). You can now use ServerPilot to manage your
server and deploy applications on it.

Full script
~~~~~~~~~~~

.. literalinclude:: examples/serverpilot.py
    :language: python

.. meta::
    :labels: serverpilot cloud-init python shade
