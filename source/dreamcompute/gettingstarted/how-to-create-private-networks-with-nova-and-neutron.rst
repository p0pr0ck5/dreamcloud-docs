===============================================================
How to create and manage private networks with Nova and Neutron
===============================================================

Private networking is a useful feature that customers may wish to utilize
for advanced cloud setups.  For a description of private networking and how to
enable it, please review the `What is private networking`_ article.  In this
article, the decisions needed for adding a private network, and how to add the
private network while using the `DreamCompute dashboard <https://iad2.dreamcompute.com>`_
are covered.

Private network options
~~~~~~~~~~~~~~~~~~~~~~~

Once Support has confirmed the network quota has been adjusted, the private
network can be added.  Please review the below options to determine settings
for the private network.

Network block
-------------

There are various private network blocks that are available for use with
private networks, and are specified in `CIDR <https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing>`_
format.  Common examples of this are 10.0.0.0/24 or 192.168.0.0/24, however
there are `other networks <https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces>`_
to choose from as well.  In the example below, we will use 10.0.0.0/24.

DHCP
----

When a subnet is defined, DHCP can be set enabled or disabled, and can be
changed later if desired.  When DHCP is enabled, newly created instances will
run cloud-init at start and detect it, and therefore determine it isn't
necessary to hard-code network settings into the operating system.  If it is
disabled, then these settings will be hard-coded.  Having DHCP enabled can help
with creating snapshots and new instances from those snapshots, as the
snapshots won't have hard-coded network configs in them.  However, older
versions of cloud-init will fail to boot entirely when DHCP is enabled.

As of October 2016, only Centos 6, and all Ubuntu versions have a version of
cloud-init that supports DHCP.  If you plan to use a different operating
system, please consider disabling DHCP.

Adding the private network
~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Create a network:

    .. code-block:: console

        [user@localhost]$ neutron net-create private-network

This command creates a new empty network which can accept a subnet
later.  In this example the name "private-network" is given.

2. Create a subnet:

    .. code-block:: console

        [user@localhost]$ neutron subnet-create private-network 10.0.0.0/24 \
                          --name private-network --dns-nameserver 8.8.8.8 \
                          --dns-nameserver 8.8.4.4 --disable-dhcp

This command creates a new subnet on top of the network created above.
Depending on decisions made about `DHCP`_ and the
`network block`_, a different CIDR and/or the flag
--enable-dhcp can be specified.  In this example the subnet is named
"private-network" the same as the network, and google DNS servers
specified.

3. Create a router:

    .. code-block:: console

        [user@localhost]$ neutron router-create private-router

This command creates a new router with a default configuration.  In this
example the name "private-router" is given.

4. Create a router interface:

    .. code-block:: console

        [user@localhost]$ neutron router-interface-add private-router private-network

This command adds an interface to the router to the private network.

5. Set the router gateway:

    .. code-block:: console

        [user@localhost]$ neutron router-gateway-set private-router public

This command sets the router gateway to the public network, to allow
it access to the internet.

This completes the process of adding a private network to the account. The
example commands below show how to select the private network and add a floating
IP address.

1. Determine flavor, security group, image, keypair, and network ID:

    .. code-block:: console

        [user@localhost]$ nova flavor-list
        [user@localhost]$ nova secgroup-list
        [user@localhost]$ nova image-list
        [user@localhost]$ nova keypair-list
        [user@localhost]$ neutron net-list

The above commands will output the available flavors, security groups,
images, keypairs, and the networks available.  Select the necessary
options for creating the instance.  For the network, the long ID is
needed in place of the given name.

2. Create an instance:

    .. code-block:: console

        [user@localhost]$ nova boot --flavor gp1.semisonic --security-group default \
                          --image Ubuntu-16.04 --nic net-id=LONG-NETWORK-UUID-HERE \
                          --key-name KEYNAME INSTANCENAME

The above command creates a semisonic size instance, using the default
security group and the Ubuntu 16.04 operating system image.  The
remaining values will vary per tenant, and will need to be specified
instead.  The LONG-NETWORK-UUID-HERE is the ID given from
"neutron net-list", the KEYNAME from "nova keypair-list" and the
instance name any name desired for the instance.

.. _`DHCP`: #dhcp
.. _`network block`: #network-block
.. _`What is private networking`: 229789688-What-is-private-networking-

.. meta::
    :labels: network
