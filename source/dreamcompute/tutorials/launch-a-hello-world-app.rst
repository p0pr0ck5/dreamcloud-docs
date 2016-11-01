=================================================================
How to deploy a simple python wsgi application on DreamHost Cloud
=================================================================

Introduction
~~~~~~~~~~~~

.. Note::

    This article assumes you are running Linux or Mac OS

In this article, you learn to deploy a simple web application on
`DreamCompute <http://dreamhost.com/cloud/compute/>`_, a public cloud powered
by OpenStack,
using the popular automation tool, `Ansible <http://www.ansible.com>`_. To get
started:

1. Sign up for a `DreamCompute account <https://signup.dreamhost.com/compute/>`_
2. `Create an SSH key
   <https://help.dreamhost.com/hc/en-us/articles/214843617-How-to-upload-an-SSH-key-via-the-web-UI>`_
3. Install a recent version of `Python <https://www.python.org/downloads/>`_ (2.7+)
4. Create a virtualenv
5. Install the `helloworld` Python package in this repository, ``shade``, and
   ``ansible``:

.. code-block:: console

    [user@localhost]$ virtualenv venv && . venv/bin/activate
    [user@localhost]$ git clone https://github.com/ryanpetrello/dreamcompute-hello-world.git hello-world
    [user@localhost]$ cd hello-world && pip install -e . shade ansible

Python and WSGI
~~~~~~~~~~~~~~~

This tutorial describes how to deploy a very simple Python WSGI application
that responds to all HTTP requests with ``Hello, World!`` WSGI is a Python
protocol that allows a web server to interface with a Python callable (or
function) to handle each request. Our function looks like this:

.. code-block:: python

    def application(environ, start_response):
          data = "Hello, World!\n"
          start_response("200 OK", [
              ("Content-Type", "text/plain"),
              ("Content-Length", str(len(data)))
          ])
          return data

but you could replace this Python code and the web server with your own
written in PHP, Ruby, Javascript, or any other language.

Deploying with Ansible
~~~~~~~~~~~~~~~~~~~~~~

This article uses Ansible to create a `repeatable "playbook" of commands
<https://github.com/ryanpetrello/dreamcompute-hello-world/blob/master/playbooks/deploy.yml>`_
to set up the Hello World application on a DreamCompute server. These commands
include:

* Launching a server in DreamCompute and assigning a public IP address to it
* Installing a web server (this tutorial uses `gunicorn <http://gunicorn.org>`_)
* Installing our example Python application and configuring it to handle
  requests

In order for Ansible to run DreamCompute API calls on your behalf, you need
to download a small shell script that sets up your API credentials called an
openrc file, read `the tutorial about openrc files
<228047207-How-to-download-your-DreamCompute-openrc-file>`__ for information on
how to download and use it.
After the shell script downloads, open your
computer's command line and run the following (substituting the actual
location of your downloaded file).

.. code-block:: console

    [user@localhost]$ source /path/to/downloaded/file/dhc123456789-openrc.sh

You are prompted for a password - it's the one you use to log in to the
DreamCompute Dashboard.

At this point you should be ready to deploy your application. Do so by running
the following commands:

.. code-block:: console

    [user@localhost]$ chmod 600 /path/to/keyname.pem
    [user@localhost]$ ansible-playbook -vvvv -i "localhost," playbooks/deploy.yml --extra-vars "key_name=keyname private_key=/path/to/keyname.pem"

You need to substitute the ``keyname`` key name value for the actual name
you chose earlier, and you also need to replace ``/path/to/keyname.pem``
with the actual path to the PEM file you downloaded.

If all is well, you are greeted with an instructional message:

    Visit http://1.2.3.4/ in your browser!

Example Server Architecture
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you ``ssh`` into your newly created server:

.. code-block:: console

    [user@localhost]$ ssh -i /path/to/keyname.pem user@1.2.3.4

you find a variety of processes running in the following configuration:

.. code::

    HTTP Request ──> <Production/Proxy Server>, nginx (1.2.3.4:80)
                      │
                      │   <supervisord> (monitors and keeps gunicorn processes running)
                      ├── <WSGI Server> gunicorn Instance (/tmp/gunicorn.sock)
                      ├── <WSGI Server> gunicorn Instance (/tmp/gunicorn.sock)
                      ├── <WSGI Server> gunicorn Instance (/tmp/gunicorn.sock)
                      ├── <WSGI Server> gunicorn Instance (/tmp/gunicorn.sock)

``supervisord`` is installed and is used to manage multiple ``gunicorn`` worker
processes, each of which is bound to a Unix domain socket (though you could
also configure them to bind to a TCP port). ``NGINX`` listens on port 80 and
balances incoming HTTP requests across the gunicorn workers processes.

.. meta::
    :labels: ansible python
