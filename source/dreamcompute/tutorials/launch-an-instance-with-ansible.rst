================================================
How to launch a DreamCompute server with Ansible
================================================

Ansible is a configuration management tool with built in OpenStack support.
Which means it's easy to use it to deploy servers and manage them on
DreamCompute using Ansible. In this tutorial we're going to explain how to
install Ansible and use it to launch a simple server on DreamCompute.

Installation
~~~~~~~~~~~~

For this tutorial we're going to use Ubuntu 16.04 (Xenial Xerus) as our host to
run Ansible playbooks from. First we need to install a few packages.
On Ubuntu 16.04 that can be done with one simple command.

.. Note::

    If you are not using Ubuntu 16.04, you can read
    http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip
    to figure out how to install ansible on your system

.. code-block:: console

    [user@localhost]$ sudo apt-get install -y ansible python2.7 python-virtualenv python-pip

Type in your password for sudo (If you have one). It then will download some
packages and install them. Next we need to install shade in a virtual
environment as Ansible depends on it.

.. code-block:: console

    [user@localhost]$ virtualenv -p /usr/bin/python2 venv && source venv/bin/activate && pip \
          install shade

Now you have shade installed and are ready to start writing Ansible
playbooks.

Writing a Ansible Playbook to Launch a Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Make a file named **launch-server.yaml**, that will be our playbook (Ansible's
term for a yaml file that Ansible can interpret and use to perform some tasks).

The first part of your playbook is a list of hosts that your playbook will run
on, we only have one, localhost.

.. literalinclude:: examples/makeserver.yaml
    :start-after: # hosts the playbook runs on
    :end-before: # List of tasks

Then we need to define a list of tasks to perform in this playbook. We will
only have one that launches an Ubuntu Xenial server on DreamCompute.

.. literalinclude:: examples/makeserver.yaml
    :start-after: # List of tasks
    :end-before: # This lets us define an server

Now we need to use the "os_server" module. This lets us define what we want our
server to look like in DreamCompute.

.. literalinclude:: examples/makeserver.yaml
    :start-after: # This lets us define an server
    :end-before: # Cloud authentication information

Now we have to tell it how to authenticate to DreamCompute so that it can
create a server there. use the following example, substituting **{username}**
with your DreamCompute username, **{password}** with your DreamCompute
password, and **{project}** with your DreamCompute project.

.. literalinclude:: examples/makeserver.yaml
    :start-after: # Cloud authentication information
    :end-before: # VM details

Now we can actually define what our server should look like.

.. literalinclude:: examples/makeserver.yaml
    :start-after: # VM details

Lets break down the previous few lines

* **state** is the state of the server, possible values are

    * present
    * absent

* **name** is the name of the server to create
* **image** is the image to boot the server from, possible values are

    * Name of an image: Ubuntu-16.04, CentOS-7, etc
    * ID of an image: 12f6a911-00a2-42eb-8712-d930da2da81f

    .. Note::

        The list of public images can be found in the web UI at
        https://iad2.dreamcompute.com/project/images/

* **key_name** is the public key to add to the server once it is created. This
  can be any key you have added to DreamCompute.
* **flavor** is the flavor of server to boot, this defines how much RAM and CPU
  your server will have. Possible values are

    * Name of a flavor: gp1.semisonic
    * ID of a flavor: 50, 100, 200, etc

* **network** is the network to put your server on. In our case it is the
  "public" network, but if you have private networking enabled, it could be
  different

    * Name of a network: public
    * ID of a network: e098d02f-bb35-4085-ae12-664aad3d9c52

* **wait** is whether or not to wait for the server to create before
  continuing. Possible values are

    * yes
    * no

Running the Ansible Playbook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Ansible playbook can be run with the following command:

.. code-block:: console

    [user@localhost]$ ansible-playbook launch-server.yaml

You should see output like

.. code::

    PLAY [localhost]
    ***************************************************************

    TASK [setup]
    *******************************************************************
    ok: [localhost]

    TASK [launch an Ubuntu server]
    ***********************************************
    changed: [localhost]

    PLAY RECAP
    *********************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0

Now if you check the `web UI
<https://iad2.dreamcompute.com/project/instances/>`_ you should see a instance
named "ansible-vm1"

Full Playbook
~~~~~~~~~~~~~

.. literalinclude:: examples/makeserver.yaml

.. meta::
    :labels: ansible
