===================================================================
How to deploy and configure PostgreSQL on DreamCompute with Ansible
===================================================================

Ansible is a configuration management tool with built-in support for
DreamCompute. This tutorial describes how to create a new Ubuntu
server on DreamHost Cloud, install and configure PostgreSQL in one
pass.

The objective is to create one PostgreSQL server on the latest Ubuntu
LTS (16.04, also known as Xenial Xerus). We're not going to configure
replication or other fancy things: only a single database server with
Ubuntu's default security configurations.

.. Note::

   Read `how to install and configure Ansible for DreamCompute
   <925908-How-to-launch-a-DreamCompute-server-with-Ansible>`_ if you
   have not used Ansible before.

Prepare the playbook to launch a new DreamCompute cloud server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First thing to do is to get the playbook ready to create a new server
on DreamHost Cloud and setup the private SSH key to use to manage it.


.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-1
     :end-before: step-2

Next step is to define what the new Ubuntu server will have to look
like:

.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-2
     :end-before: step-3

The task above will connect to the cloud named `iad2` in your
openstack/clouds.yaml configuration file, tell OpenStack Nova to
create a new instance called `postgres01` based on the Glance image
called Ubuntu-16-04, picking the flavor with 2GB RAM, and booting from
a new 40GB volume. The task will also add the public key associated with
`mykey` to your server: make sure the corresponding private key is the
one specified on the `private_key` variable.  Finally the task waits
for the new machine to be created before registers the name
`pgdb_server` and moving on to the next steps.

Once the server has been created, Ansible needs to store some basic
facts about it. The next tasks are all about getting to know the new
server:

.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-3
     :end-before: step-4

Gather the public IP address of the server and add it to the Ansible
hosts catalog. The `add_host` task creates a new entry assigning the
IP of the new server to the Ansible group `pgsql` and configure the
SSH connection with the default username for your image, and set the private SSH key
specified at the beginning.

Make the new Ubuntu 16.04 compatible with Ansible
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All the information necessary to install PostgreSQL on the new server
are now in place. All next steps are going to be executed on the newly
created DreamHost Cloud server.

.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-4
     :end-before: step-5

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
   `pgsql`.


Install PostgreSQL and create a new database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The new server is ready to install PostgreSQL server and create a
database in it.

.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-5
     :end-before: step-6

This task instructs Ansible to connect to all hosts in the `pgsql`
group, use python2.7 interpreter and install PostgreSQL server, Python
Psycopg interface and libraries. It also sets the variables to be used
to create the new postgres DB using the Ansible module for PostgreSQL:

.. literalinclude:: examples/postgresql_server.yaml
     :start-after: step-6

Running the Ansible Playbook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Ansible playbook can be run with the following command:

.. code-block:: console

    [user@localhost]$ ansible-playbook postgresql-server.yaml

You'll soon see on `DreamHost Cloud web UI
<https://iad2.dreamcompute.com/project/instances/>`_ the new instance
and the new volume. Login into the new machine and check that the
database is really there:

.. code-block:: console

    [user@server]$ psql -h localhost -d mydb -U dreamer
    Password for user dreamer:
    psql (9.5.3)
    SSL connection (protocol: TLSv1.2, cipher:
    ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.

    mydb=> \l
                                      List of databases
       Name    |  Owner   | Encoding  |   Collate   |    Ctype    | Access privileges
    -----------+----------+-----------+-------------+-------------+-----------------------
     mydb      | postgres | UTF8      | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
               |          |           |             |             | postgres=CTc/postgres+
               |          |           |             |             | dreamer=CTc/postgres
     postgres  | postgres | SQL_ASCII | C           | C           |
     template0 | postgres | SQL_ASCII | C           | C           | =c/postgres          +
               |          |           |             |             | postgres=CTc/postgres
     template1 | postgres | SQL_ASCII | C           | C           | =c/postgres          +
               |          |           |             |             | postgres=CTc/postgres
    (4 rows)

    mydb=>

Easy! Next steps are about securing the newly created server and
setting the proper security groups so that applications can connect to
the SQL demon.

Full Playbook
~~~~~~~~~~~~~

.. literalinclude:: examples/postgresql_server.yaml

.. meta::
    :labels: ansible postgresql
