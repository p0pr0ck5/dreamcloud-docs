========================================================
How to use Cloud-Init on DreamCompute to deploy fail2ban
========================================================

Cloud-init is the package that handles early initialization of a cloud
instance: it can be used to automatically configure a new server as
soon as it starts, without logging into it. This tutorial shows how
install fail2ban in any Linux distribution available on DreamHost
Cloud.

Cloud-init can be used either the web dashboard, or using the command
line.

Using the Web Dashboard
~~~~~~~~~~~~~~~~~~~~~~~

Open the web dashboard in a browser, then navigate to the instance tab
by clicking on Compute > Instances on the left side of the browser.
Click the `Launch Instance button
<https://iad2.dreamcompute.com/project/instances/launch>`_, give your
instance a name, boot from a Linux-based image (Ubuntu, Debian, , select the default
security group, then click the "Post Creation" Button on the far right
of the "Launch Instance" page. Here you can either upload a file to be
used with Cloud-Init or just input the contents of the file into the
web UI. Copy and paste the text below into the text box.

.. code::

    #cloud-config
    package_upgrade: true
    packages:
     - fail2ban

    # download a custom config file hosted on a public server
    runcmd:
     - [ wget, "https://gist.github.com/smaffulli/de8f6eb097fdedad0e8c3487953967ff", -O, /etc/fail2ban/jail.local ]
     - service fail2ban restart

.. note::

   Make sure to include the line `#cloud-config`, as that is not a comment.


Visit `How to launch and manage instances with the DreamCompute dashboard`_ for
more information on how to launch an instance with the web dashboard.

Using the OpenStack Client
~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create an DreamCompute instance and pass it a script for cloud init to
execute at creation using the openstack commandline client. First you need a file to
with the script that you want to run, we'll call it cloud-init.sh. We want
Cloud-Init to install fail2ban, so the contents of cloud-init.sh should be:

.. code::

    #cloud-config
    package_upgrade: true
    packages:
     - fail2ban
    # download a custom config file hosted on a public server
    runcmd:
     - [ wget, "http://dhurl.org/2gy", -O, /etc/fail2ban/jail.local ]
     - service fail2ban restart

The command to create an instance that runs cloud-init.sh at creation is:

.. code::

    [user@localhost]$ openstack server create --image Ubuntu-16.04 --flavor \
        gp1.semisonic --security-group default --key-name stef --user-data \
        cloud-init.sh newserver

Visit `How to launch a virtual server on DreamCompute`_ for more information
on how to launch an instance from the command line

Writing your own script to be used with Cloud-Init
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can put anything in a shell script for Cloud-Init to execute while creating
your instance, although you should only run commands that don't take user
input. Cloud-Init can be used to install packages at instance creation, but if
you want to install more than a couple things that you need to manage closely,
it is recommended you use a configuration management tool like Ansible or
Puppet as they provide more control over your system.

.. meta::
    :labels: cloud-init security

.. _How to launch a virtual server on DreamCompute: 216511617

.. _How to launch and manage instances with the DreamCompute dashboard: 215912848
