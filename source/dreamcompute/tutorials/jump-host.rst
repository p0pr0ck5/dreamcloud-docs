=====================================================
How to SSH to an Instance Without a Public IP Address
=====================================================

Overview
~~~~~~~~

If you have private networking enabled in DreamCompute, you may use
it to expose some servers to the
internet while limiting access to others on the private network. For
example, you may want your web server exposed on port 80 or 443 while
keeping your database access restricted to only the private network.
One complication of this configuration is how to access your private
instances without a floating IP address. The answer to that is to use
a jump host.

How it works
~~~~~~~~~~~~

A jump (or bastion) host acts as a middleman for all traffic to your
non-public instances and is easy to set up using SSH and its
configuration options. The first thing you want to do is to make sure
you have an instance running with a floating IP address. This could be
any DreamCompute instance with a floating IP, but you'd typically use
a dedicated jump host or your web server in the example above. Ensure
you can SSH into that machine with your SSH key. If you need
instructions for that, check out `how to upload SSH keys in
DreamCompute`_.

Configuring a Jump Host in DreamCompute
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following example shows a setup of an SSH config file
(~/.ssh/config) on a local machine so that it can connect first to a
jump host called “jump”. Change $username to the default user for your image,

.. include:: common/usernames.rst
    :start-line: 2

#. Enter the following:

    .. code::

        Host jump
        HostName X.X.X.X #Replace with your Floating IP Address
        User $username
        IdentityFile ~/.ssh/id_rsa.pub

    * Replace "X.X.X.X" with the floating IP address for your instance in the
      HostName option.
    * Be sure you’re using the correct public key.

#. Ensure you can log into your jump host with SSH:

    .. code-block:: console

        [user@localhost]$ ssh jump

#. Once you verify that it works, update the SSH config to send all
   traffic from your machine to the private DreamCompute network
   through a proxy on the jump host by entering the following:

    .. code::

        Host 10.10.10.*
        ProxyCommand ssh jump -W %h:%p
        User $username
        IdentityFile ~/.ssh/dreamcompute.pem

    .. note::

        * This example uses a separate key for private instances
          rather than the jump host, which allows you to keep both
          keys on your machine - the jump host doesn't need the
          private key for the other instances. This provides an extra
          level of security.
        * The -W option forwards all stdin and stdout to the specified
          host and port.

#. To connect to your private instances from a local machine, SSH to
   the private IP:

    .. code-block:: console

        [user@localhost]$ ssh 10.10.10.5

.. _how to upload SSH keys in DreamCompute: 214843617-How-to-upload-an-SSH-key-via-the-web-UI

.. meta::
    :labels: ssh nova windows mac linux desktop dashboard
