===============================================================
How to deploy multinode WordPress on DreamCompute using Ansible
===============================================================

Why Ansible?
~~~~~~~~~~~~

Ansible is a configuration management tool that has OpenStack integration built
in. This means you can use ansible not only to configure your servers, but when
running them on OpenStack, ansible can create and destroy servers for you.
Deploying a multinode WordPress site is tedious and long as you can see in
`the manual tutorial
<228715787-How-to-deploy-multinode-WordPress-on-DreamCompute>`__
. You can automate all those steps easily with Ansible following the
instructions below.

.. include:: common/install_ansible_debian_or_ubuntu.rst

Downloading the playbook
~~~~~~~~~~~~~~~~~~~~~~~~

Download the Ansible playbook by cloning the OpenStack Ops (osops) repository:

.. code-block:: console

    [user@localhost] git clone https://github.com/squidboylan/osops-tools-contrib.git

Configuring the playbook
~~~~~~~~~~~~~~~~~~~~~~~~

Navigate to the Ansible playbook by running:

.. code-block:: console

    [user@localhost] cd osops-tools-contrib/ansible/lampstack

Edit the vars/dreamhost.yml file and make the following changes:

* Set the username variable

    .. code-block:: yaml

        username: "DreamCompute username goes here",

* Set the project_name variable

    .. code-block:: yaml

        project_name: "DreamCompute project name goes here",

    .. Note::

        Your project name can be found in your `OpenRC file
        <228047207-How-to-download-your-DreamCompute-openrc-file>`__

* Set the public_key_file variable

    .. code-block:: yaml

        public_key_file: "/home/username/.ssh/id_rsa.pub",

  Change the path to the path of your public key

Running the playbook
~~~~~~~~~~~~~~~~~~~~

In order to run the playbook, run the following command:

.. code-block:: console

    (venv)[user@localhost] ansible-playbook -e \
        "env=dreamhost action=apply password=yourpassword" site.yml

This will create your WordPress site, after it runs, visit the IP address of
your balancer, and if everything worked you should see a WordPress site. The
full run of the playbook may take 8 - 12 minutes.

In order to delete the site, run the following:

.. code-block:: console

    (venv)[user@localhost] ansible-playbook -e \
        "env=dreamhost action=destroy password=yourpassword" site.yml

Extra configuration
~~~~~~~~~~~~~~~~~~~

There are several configuration changes that can be made to the playbook to
modify the WordPress site it creates. Take a look at the "vars/dreamhost.yml"
file:

* ``stack_size`` is the variable that defines how many servers to use for your
  site. The number of webservers you have is stack_size-2.
* ``flavor_name`` is the flavor of server to use for the servers, change this
  to whatver server flavor you want.
* ``volume_size`` is the size of the volume to put your MySQL database on. A
  bigger site with more data will need a bigger volume.
* ``wp_theme`` is the WordPress theme to use with the site.
* ``wp_title`` is the title of the WordPress site

The "group_vars/all.yml" file also has some configuration:

* ``db_user`` is the database user that wordpress will use
* ``db_pass`` is the password for the database user that wordpress will use, it
  also gets used as the admin password for the website. The default setting
  will generate a random password using ascii characters that is 8 letters long
  and store it in /tmp/sqlpassword

.. meta::
    :labels: ansible wordpress apache mysql
