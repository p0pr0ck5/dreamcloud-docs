===========================================
Load Balancing on DreamCompute with HAProxy
===========================================

Introduction
~~~~~~~~~~~~

Load balancers are designed to host multiple applications behind the
same IP address to save cost and to allow applications to horizontally
scale out instances running on multiple virtual machines.
One of the most popular open source load balancers is
`HAProxy <http://www.haproxy.org/>`_. It is well supported,
featureful, high performing, and widely available.

In this tutorial, we will be creating a virtual machine inside of
DreamCompute that acts as a load balancer with HAProxy. We’ll be using
Ubuntu 14.04 as the base operating system image for our load balancer.

Prepping Your Virtual Machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create a virtual machine in DreamCompute using your favorite method, a
*subsonic* flavor should be enough. Make sure to assign an SSH key and
allow access to  HTTP (port 80) and SSH (port 22) in the Access &
Security group. Associate a public IPv4 "floating" IP address to this
machine.

.. figure:: images/HAProxy_instances_dash.fw.png

1. Click the **More** dropdown tab on in the Actions column of your
   instance list.

.. figure:: images/HAProxy_floating_ip.fw.png

2. Click **Associate Floating IP**.

   .. note::

        This is only necessary if you have private networking enabled

    *The following dialog box appears:*

.. figure:: images/HAProxy_manage_floating_ip.fw.png

3. Allocate a floating IP with the port associated with the private IP
   address of your load balancer virtual machine, and then click the
   **Associate** button.

    * Once you've associated the IP address, give DreamCompute a few
      seconds to complete the association, and then you should be able
      to SSH into your instance via its IPv4 address.

   .. note::

        This is only necessary if you have private networking enabled

.. figure:: images/HAProxy_SSH1.fw.png

4. SSH into your instance via its IPv4 address (the default user is
ubuntu, you can't login as root).

Now, we’re ready to get HAProxy installed and configured.

Install HAProxy
~~~~~~~~~~~~~~~

Updating Packages
-----------------

Before installing HAProxy, its important to be sure that your
operating system is up to date with the latest security fixes and
packages available from Ubuntu. While logged in as ubuntu, simply
run:

.. code-block:: console

    [user@server]$ sudo apt-get upgrade

Follow the prompts, and allow apt to update your system.

Installing HAProxy
------------------

.. code-block:: console

    [user@server]$ sudo apt-get install haproxy

Configure HAProxy
~~~~~~~~~~~~~~~~~

For the purposes of our tutorial, we're going to assume that you have
two applications that you would like to deploy behind HAProxy. The
first application will be on a single virtual machine and the second
application will be horizontally scaled across two virtual machines.

When we are complete with our configuration, our deployment will look
something like this:

.. code::

    app1.domain.io
        |          +-------------+   10.10.10.2   +---------------+
        +--------->|             |--------------->|      app1     |
                   |   HAProxy   |                +---------------+
        +--------->|             |---+
        |          +-------------+   | 10.10.10.3 +---------------+
    app2.domain.io                   +----------->|    app2-a     |
                                     |            +---------------+
                                     |
                                     | 10.10.10.4 +---------------+
                                     +----------->|    app2-b     |
                                                  +---------------+

To support this deployment, edit your /etc/haproxy/haproxy.cfg:

.. code::

    global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    maxconn 4096
    user haproxy
    group haproxy
    daemon

    defaults
    log    global
    mode    http
    option    httplog
    option    dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

    option forwardfor
    option http-server-close
    stats enable
    stats auth admin:stats
    stats uri /haproxyStats

    frontend http-in
    bind \*:80
    option forwardfor

    # Define hosts
    acl host_app_one hdr(host) -i app1.domain.io
    acl host_app_two hdr(host) -i app2.domain.io

    # Figure out which one to use
    use_backend app_one_cluster if host_app_one
    use_backend app_two_cluster if host_app_two

    backend app_one_cluster
    balance leastconn
    option httpclose
    server node1 10.10.10.2:80 cookie A check

    backend app_two_cluster
    balance leastconn
    option httpclose
    server node1 10.10.10.3:80 cookie A check
    server node2 10.10.10.4:80 cookie A check

Next, you'll need to activate HAProxy by setting ENABLED to 1 in
/etc/default/haproxy. Finally, you can run HAProxy:

.. code-block:: console

    [user@server]$ sudo service haproxy restart

Assuming that you have configured your DNS to point app1.domain.io and
app2.domain.io to your public IP address, you should be able to
access your applications via HAProxy. Congratulations!

.. meta::
    :labels: apache haproxy debian ubuntu
