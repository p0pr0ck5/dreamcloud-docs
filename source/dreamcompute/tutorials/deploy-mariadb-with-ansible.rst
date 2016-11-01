================================================================
How to deploy and configure MariaDB in DreamCompute with Ansible
================================================================

Ansible is a configuration management tool with built-in support for
DreamCompute. This tutorial describes how to create a new Ubuntu
server on DreamHost server, install and configure MariaDB in one pass.
The objective is to create one MariaDB server on the latest Ubuntu
LTS (16.04, also known as Xenial Xerus). We're not going to configure
replication for MariaDB, only a single database server with basic
security configurations.

.. Note::

   Read `how to install and configure Ansible for DreamCompute
   <925908-How-to-launch-a-DreamCompute-server-with-Ansible>`_ if you
   have not used Ansible before.

Prepare the playbook to launch a new DreamCompute cloud server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First thing to do is to get the playbook ready to create a new server
on DreamHost Cloud and setup the private SSH key to use to manage it.

.. code::

    - hosts: localhost
      connection: local
      vars:
        private_key: /the/path/to/your/private/ssh/key

Next step is to define what the new Ubuntu server will have to look
like:

.. code::

    - name: create a Ubuntu server
      os_server:
            cloud: iad2
            name: mariadb01
            state: present
            image: Ubuntu-16.04
            flavor_ram: 2048
            key_name: mykey
            boot_from_volume: True
            volume_size: 10
            network: public
            wait: yes
      register: mariadb_server

The task above will connect to the cloud named **iad2** in your
openstack/clouds.yaml configuration file, tell OpenStack Nova to
create a new instance called *mariadb01* based on the Glance image
called Ubuntu-16-04, picking the flavor with 2GB RAM, and booting from
a new 10GB volume. The task will also add the public key associated with
`mykey` to your server: make sure the corresponding private key is the
one specified on the `private_key` variable.  Finally the task waits
for the new machine to be created before registers the name
`mariadb_server` and moving on to the next steps.

Once the server has been created, Ansible needs to store some basic
facts about it. The next tasks are all about getting to know the new
server:

.. code::

    - name: get facts about the server (including its public v4 IP
      address)
      os_server_facts:
        cloud: iad2
        server: mariadb01
      until: mariadb_server.server.public_v4 != ""
      retries: 5
      delay: 10

    - set_fact: public_v4="{{ mariadb_server.server.public_v4 }}"

    - name: add the server to our ansible inventory
      add_host: hostname={{ public_v4 }} groups=sql ansible_ssh_user=user ansible_ssh_private_key_file={{ private_key }}

Gather the public IP address of the server and add it to the Ansible
hosts catalog. The `add_host` task creates a new entry assigning the
IP of the new server to the Ansible group `sql` and configure the SSH
connection with the default user for your image, and set the private SSH key
specified at the beginning.

Make the new Ubuntu 16.04 compatible with Ansible
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All the information necessary to deploy MariaDB on the new server are
now in place: next steps are going to be executed on the new host.

.. code::

    - hosts: sql
      gather_facts: no
      tasks:

        - name: Install python2.7 since Ubuntu 16.04 doesn't ship it
          raw: "sudo apt-get update -y && sudo apt-get install -y python2.7 aptitude"

The first line specifies the group of hosts to execute tasks on. Since
Ubuntu LTS 16.04 Xenial comes only with Python3, we need to install
Python 2.x before we can continue with Ansible. The key to proceed
successfully is the line `gather_facts: no`, since that module will
fail otherwise. After that, the `raw` task will not rely on python and
execute the apt-get commands to get the latest python 2.7 package. We
also need to install the command `aptitude` in order to use the `apt`
module provided by Ansible.

.. Note::

   Using groups in Ansible inventory allows to execute the same task
   on all hosts. In this case there is only one server in the group
   `sql`.


Install MariaDB and create a new database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The new server is ready to install MariaDB server and create a
database in it.

.. code::

    - hosts: sql
      vars:
       ansible_python_interpreter: /usr/bin/python2.7

      become: True

      tasks:
        - name: Install MariaDB
          apt: name={{ item }} state=latest update_cache=yes
          with_items:
            - mariadb-server
            - python-mysqldb
        - name: Create a new db
          mysql_db: name=backend_db state=present

This task instructs Ansible to connect to all hosts in the `sql`
group, use python2.7 interpreter and install MariaDB server and Python
MySQLDB interfaces. Then use the core MySQL module to create a new
database called `backend_db`.

Running the Ansible Playbook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Ansible playbook can be run with the following command:

.. code-block:: console

    [user@localhost]$ ansible-playbook mariadb-server.yaml

You'll soon see on `DreamCompute web UI
<https://iad2.dreamcompute.com/project/instances/>`_ the new instance
and the new volume. Login into the new machine and check that the
database is really there:

.. code-block:: console

    [user@localhost]$ sudo mysql backend_db
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 46
    Server version: 10.0.25-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

    Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input
    statement.

    MariaDB [backend_db]>


Easy! Next steps are about securing the newly created server and
setting the proper security groups so that applications can connect to
the SQL demon.

Full Playbook
~~~~~~~~~~~~~

.. literalinclude:: examples/mariadb_server.yaml

.. meta::
    :labels: ansible mysql mariadb
