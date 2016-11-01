=====================================================
How to use Cloud-Init on DreamCompute to deploy NGINX
=====================================================

Cloud-Init can be used to provide a script to an instance that runs at the
instance's creation. This tutorial will show how to deploy NGINX on an Ubuntu
Trusty instance during its creation.

This can be done in either the web dashboard, or using the command line.

Using the Web Dashboard
~~~~~~~~~~~~~~~~~~~~~~~

Open the web dashboard in a browser, then navigate to the instance tab by
clicking on Compute > Instances on the left side of the browser.
Click the Launch Instance button, give
your instance a name, boot from an Ubuntu Trusty image, select the default
security group, then click the "Post Creation" Button on the far right of the
"Launch Instance" page. Here you can either upload a file to be used with
Cloud-Init or just input the contents of the file into the web UI. Copy and
paste the text below into the text box.

.. code-block:: bash

    #! /bin/bash
    apt-get install nginx -y

.. note::

    The `-y` is so that it doesn't ask for user input, it just assumes yes to
    any questions.

Visit `How to launch and manage instances with the DreamCompute dashboard`_ for
more information on how to launch an instance with the web dashboard.

Using the Nova Client
~~~~~~~~~~~~~~~~~~~~~

You can create an Ubuntu Trusty instance and pass it a script for cloud init to
execute at creation using the nova commandline client. First you need a file to
with the script that you want to run, we'll call it cloud-init.sh. We want
Cloud-Init to install NGINX, so the contents of cloud-init.sh should be:

.. code-block:: bash

    #! /bin/bash
    apt-get install nginx -y

.. note::

    The `-y` is so that it doesn't ask for user input, it just assumes yes to
    any questions.

The command to create an instance that runs cloud-init.sh at creation is:

.. code-block:: console

    [root@localhost]# nova boot --image Ubuntu-14.04 --flavor 100 --user-data \
        cloud-init.sh

Visit `How to launch a virtual server on DreamCompute`_ for more information
on how to launch an instance from the command line

Using A different Linux Distro
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you wanted to deploy NGINX on a different Linux distro, the only thing that
needs to be changed is the script that Cloud-Init runs to install NGINX.
For example if you wanted to deploy NGINX on CentOS, your file would look
something like:

.. code-block:: bash

    #! /bin/bash
    yum -y install nginx
    service nginx start

Troubleshooting
~~~~~~~~~~~~~~~

After the instance launches, you should see the NGINX welcome page when you
visit its IP in a web browser. If you do not, you should check your security
groups to see if they are blocking port 80.

Writing your own script to be used with Cloud-Init
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can put anything in a shell script for Cloud-Init to execute while creating
your instance, although you should only run commands that don't take user
input. Cloud-Init can be used to install packages at instance creation, but if
you want to install more than a couple things that you need to manage closely,
it is recommended you use a configuration management tool like Ansible or
Puppet as they provide more control over your system.

.. meta::
    :labels: cloud-init nginx

.. _How to launch a virtual server on DreamCompute: 216511617

.. _How to launch and manage instances with the DreamCompute dashboard: 215912848
