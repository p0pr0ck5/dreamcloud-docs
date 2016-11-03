=========================================================================
How to create and manage private networks with the DreamCompute Dashboard
=========================================================================

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

Using the private network
~~~~~~~~~~~~~~~~~~~~~~~~~

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

.. _`DHCP`: #dhcp
.. _`network block`: #network-block
.. _`What is private networking`: 229789688-What-is-private-networking-

.. meta::
    :labels: network
