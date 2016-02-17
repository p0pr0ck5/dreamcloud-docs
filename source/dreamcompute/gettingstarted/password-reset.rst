=======================
Resetting Your Password
=======================

DreamHost Control Panel
~~~~~~~~~~~~~~~~~~~~~~~

It's easy to reset your DreamCompute password in the DreamHost Control Panel.

* Go to the `DreamCompute <https://panel.dreamhost.com/dreamcompute>`_ page in
  the control panel.
* Click the "Login Details" button and then the "Reset My Password" button.
* Your new password will be displayed at the top of the page. Please note that
  a may take a few minutes for the change to take affect.

.. figure:: images/DreamCompute-reset-password.png

DreamCompute Dashboard
~~~~~~~~~~~~~~~~~~~~~~

You can change your password using the DreamCompute dashboard. Go to Settings
-> Change Password, in the DreamCompute Dashboard, enter your current password
and your new password.

.. figure:: images/DHC-change-password.png

Use Keystone Client to Reset Your Password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DreamCompute utilizes the `Keystone
<http://docs.openstack.org/developer/keystone/>`_ component of OpenStack for
authentication and identity management. You can reset your password using the
Keystone `API client
<http://docs.openstack.org/api/quick-start/content/>`_.

You can use the Keystone command-line client to change your password. This
assumes you already have the OS_PASSWORD variable set via the OpenStack RC
file.

.. code-block:: bash

    keystone password-update --new-password  <new-password>

**Don't have the Keystone client?** The
`OpenStack wiki
<http://docs.openstack.org/user-guide/content/install_clients.html>`_ has good
instructions for installing it (and the rest of the clients).

API Access
~~~~~~~~~~

If you don't already have an OpenStack RC file downloaded from the
Access & Security section, you can create
your own.

.. code-block:: bash

    #!/bin/bash
    export OS_AUTH_URL=https://keystone.dream.io/v2.0
    export OS_TENANT_ID=YOUR_TENANT_ID
    export OS_TENANT_NAME="dhcNUMBER"
    export OS_USERNAME="USERNAME"
    echo "Please enter your OpenStack Password: "
    read -sr OS_PASSWORD_INPUT
    export OS_PASSWORD=$OS_PASSWORD_INPUT

Set the variables.

.. code-block:: bash

    source TENANTNUM-openrc.sh

or

.. code-block:: bash

    . TENANTNUM-openrc.sh

Type in your current password when prompted.

.. meta::
   :labels: password reset
