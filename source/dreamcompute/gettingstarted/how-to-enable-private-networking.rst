===========================
What is private networking?
===========================

Private networking is a type of network in which all the servers connected
to that network are able to talk to each other and reach the rest of the
internet through a router, but the rest of the internet cannot initialize
a connection to those servers.

Private networking is ideal for running services that you wish to make
secure, as these services are unreachable from the internet and thus are
much harder to hack.

Enabling private networking
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The DreamCompute US-East 2 cluster has public networking available by default
which means every one of your servers gets allocated a public IP address when
it is created on that network.

In order to enable private networking on your account in US-East 2, please
contact support and make a request to enable it. Currently, this is unavailable
through the panel.

Price
~~~~~

Private networking in US-EAST 2 costs $5 per private network.

Private network options
~~~~~~~~~~~~~~~~~~~~~~~

Once support has confirmed the network quota has been adjusted, the private
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

Three different ways are described below to accomplish the same
goal, depending on if the dashboard or command line is preferred.

* `DreamCompute dashboard`_
* `Command line with nova and neutron`_

DreamCompute dashboard
----------------------

1. Begin creating a network:

    .. figure:: images/how-to-enable-private-networking/network-01.png

        Launch the DreamCompute dashboard, and navigate to the Network
        -> Networks menu.  Click on the "+ Create Network" button on the
        top right.

2. Enter network information:

    .. figure:: images/how-to-enable-private-networking/network-02.png

        A new window appears.  Enter a name for the network such as
        "private-network".  Then, click the "Next" button.

3. Enter subnet information:

    .. figure:: images/how-to-enable-private-networking/network-03.png

        On the Subnet tab, enter the above determined CIDR for the
        private network into the "Network Address" field.  An optional
        Subnet name can be specified if desired.  Then, click the "Next"
        button.

4. Enter subnet details:

    .. figure:: images/how-to-enable-private-networking/network-04.png

        On the Subnet Details tab, check or uncheck the "Enable DHCP"
        checkbox depending on the decision made in the `DHCP`_
        section above.  In the "DNS Name Servers" field, enter the
        values 8.8.8.8 and 8.8.4.4 on their own lines.  Finally, click the
        "Create" button.

5. Begin creating a router:

    .. figure:: images/how-to-enable-private-networking/network-05.png

        Navigate to the Network -> Routers menu.  Click on the "+ Create
        Router" button on the top right.

6. Enter router information:

    .. figure:: images/how-to-enable-private-networking/network-06.png

        A new window appears.  Enter a name for the router such as
        "private-router", and select "public" from the "External Network"
        drop-down.  Finally, click the "Create Router" button.

7. Begin adding a router interface:

    .. figure:: images/how-to-enable-private-networking/network-07.png

        Once the router is displayed, click on the routers name to navigate
        to the router details page.  Click on the "Interfaces" tab that is
        displayed on the top left.

8. Add an interface:

    .. figure:: images/how-to-enable-private-networking/network-08.png

        Click on the "+ Add Interface" button on the top right.

9. Enter interface information:

    .. figure:: images/how-to-enable-private-networking/network-09.png

        In the "Subnet" drop-down, select the private network created in
        steps #1-4 above.  Finally, click the "Add Interface" button.

This completes the process of adding a private network to the account. To
select the private network and add a floating IP address, follow the additional
steps below.

1. Begin adding an instance:

    .. figure:: images/how-to-enable-private-networking/network-10.png

        Navigate to the Compute -> Instances menu.  Click on the "Launch
        Instance" button on the top right.  Complete the "Details", "Access
        & Security" and "Post-Creation" tabs as normal.  In the "Networking"
        tab, click the "+" button to add the private network to this instance.
        Finally, click the "Launch" button to launch the instance.

2. Begin floating IP association:

    .. figure:: images/how-to-enable-private-networking/network-11.png

        In the right drop-down menu beside the instance, click the down arrow
        to expand it and select "Associate Floating IP".

3. Provision a floating IP address if needed:

    .. figure:: images/how-to-enable-private-networking/network-12.png

        If a floating IP has not yet been provisioned, click the "+" button
        to do so.  The provision window has only one "Pool" available named
        "Public" to select, and an "Allocate IP" button to complete the
        process.  Select an available floating IP from the "IP Address"
        drop-down, and the private IP address of the above instance in the
        "Port to be associated" drop-down.  Finally, click the "Associate"
        button.

4. Verify floating IP assignment:

    .. figure:: images/how-to-enable-private-networking/network-13.png

        The floating IP address assigned will appear on the Compute ->
        Instances page in the "IP Address" column.

Command line with nova and neutron
----------------------------------

1. Create a network:

    .. code:: console

        $ neutron net-create private-network

This command creates a new empty network which can accept a subnet
later.  In this example the name "private-network" is given.

2. Create a subnet:

    .. code:: console

        $ neutron subnet-create private-network 10.0.0.0/24 --name private-network \
          --dns-nameserver 8.8.8.8 --dns-nameserver 8.8.4.4 --disable-dhcp

This command creates a new subnet on top of the network created above.
Depending on decisions made about `DHCP`_ and the
`network block`_, a different CIDR and/or the flag
--enable-dhcp can be specified.  In this example the subnet is named
"private-network" the same as the network, and google DNS servers
specified.

3. Create a router:

    .. code:: console

        $ neutron router-create private-router

This command creates a new router with a default configuration.  In this
example the name "private-router" is given.

4. Create a router interface:

    .. code:: console

        $ neutron router-interface-add private-router private-network

This command adds an interface to the router to the private network.

5. Set the router gateway:

    .. code:: console

        $ neutron router-gateway-set private-router public

This command sets the router gateway to the public network, to allow
it access to the internet.

This completes the process of adding a private network to the account. The
example commands below show how to select the private network and add a floating
IP address.

1. Determine flavor, security group, image, keypair, and network ID:

    .. code:: console

        $ nova flavor-list
        $ nova secgroup-list
        $ nova image-list
        $ nova keypair-list
        $ neutron net-list

The above commands will output the available flavors, security groups,
images, keypairs, and the networks available.  Select the necessary
options for creating the instance.  For the network, the long ID is
needed in place of the given name.

2. Create an instance:

    .. code:: console

        $ nova boot --flavor gp1.semisonic --security-group default --image Ubuntu-16.04 \
          --nic net-id=LONG-NETWORK-UUID-HERE --key-name KEYNAME INSTANCENAME

The above command creates a semisonic size instance, using the default
security group and the Ubuntu 16.04 operating system image.  The
remaining values will vary per tenant, and will need to be specified
instead.  The LONG-NETWORK-UUID-HERE is the ID given from
"neutron net-list", the KEYNAME from "nova keypair-list" and the
instance name any name desired for the instance.

.. _`DHCP`: #dhcp
.. _`network block`: #network-block
.. _`DreamCompute dashboard`: #dreamcompute-dashboard
.. _`Command line with nova and neutron`: #command-line-with-nova-and-neutron


.. meta::
    :labels: network
