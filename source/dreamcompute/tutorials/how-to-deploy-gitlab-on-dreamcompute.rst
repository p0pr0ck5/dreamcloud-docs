===================================
How to setup GitLab on DreamCompute
===================================

Setting up
----------
Before you can put GitLab on a DreamCompute server, you must first have an
server running. I recommend launching an Ubuntu 16.04 server. You can launch a
server using the Web UI.  Your server must have at least 2GB of RAM in order
to run GitLab properly, although more is better (especially if you have lots
of users). GitLab will install if you have less than 2GB of RAM, but you will
run into weird errors, like 500 errors when you visit the site. For more info
about system requirements, visit
http://docs.gitlab.com/ee/install/requirements.html#hardware-requirements . It
is also recommended that you run GitLab on a volume backed instance, for more
info on this, visit our `tutorial on launching instances <215912848>`_

Installing GitLab
-----------------
Once you have an Ubuntu 16.04 server running, ssh in with

.. code-block:: console

    [user@localhost]$ ssh user@floatingip

replacing "floatingip" with the ip address of your server, then run

.. code-block:: console

    [user@server]$ sudo apt install postfix

In the postfix installer, select "Internet site". In order to get the package
for GitLab and install it run the following.

.. code-block:: console

    [user@server]$ curl -LJO https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/xenial/gitlab-ce_8.8.4-ce.0_amd64.deb/download
    [user@server]$ sudo dpkg -i gitlab-ce_8.8.4-ce.0_amd64.deb

.. Note::

    The url that you curl may be different, go to
    https://packages.gitlab.com/gitlab/gitlab-ce to find the newest version of
    GitLab.

Configuration
-------------

.. code-block:: console

    [user@server]$ sudo gitlab-ctl reconfigure

Congrats, you are now running GitLab. The last thing you need to do is visit
gitlab in a web browser by going to the ip of your server, then use the webpage
to change the password for root.

.. meta::
    :labels: gitlab
