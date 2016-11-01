======================================================
Step-by-step guide to set up Trellis on DreamCompute
======================================================

Trellis
~~~~~~~

In this tutorial we are going to use `Trellis
<https://roots.io/trellis/>`_
to install a very solid modern LEMP stack on DreamCompute. This LEMP stack by
Roots is great to run WordPress websites and works well with `Bedrock
<https://roots.io/bedrock/>`_
, the modern WordPress stack.

Modern LEMP Stack
~~~~~~~~~~~~~~~~~

Trellis is a set of Ansible playbooks that help you setup a full local,
staging and development environment for your WordPress project. With it you
will have a WordPress ready server running locally or remotely (intention
in this tutorial) with:

* Nginx
* MariaDB (MySQL drop-in replacement)
* PHP 7 (+ extensions)
* Composer
* WP-CLI
* sSMTP
* Memcached
* Fail2ban
* ferm (firewall)
* Mailhog

Locally it works with an automated Vagrant setup and remotely for staging
it sets you up with a fully fledged WordPress server. It also helps you
deploy your WordPress site once you are ready to do so.


Requirements
~~~~~~~~~~~~

There are several `requirements
<https://roots.io/trellis/docs/installing-trellis/>`_
to work with Trellis locally and to be able to work on the remote server:

* Ansible 2.0.2.0
* Virtualbox >= 4.3.10
* Vagrant >= 1.5.4
* vagrant-bindfs >= 0.3.1 (Windows users may skip this if not using vagrant-winnfsd for folder sync)
* vagrant-hostmanager

**NOTES**

Ansible is both needed for setting up a remote server for staging or
production on DreamCompute. Git and an accessible online repository will also be
needed as well as the latest Python 2.x version for running Ansible 2.0.2.0.


On Ubuntu most if not all of these tools can be installed using the
package manager apt-get. On OSX Homebrew and Pip are your friends. On
Windows more Linux tools have become available with the latest version
such as Bash and other needed dependencies can be installed as well using
various tools. Tougher though than on Nix systems as you can read `here
<https://roots.io/trellis/docs/windows/>`_ .

Trellis Setup
~~~~~~~~~~~~~

**NB** Taken from `Roots Trellis documentation on installing Trellis
<https://roots.io/trellis/docs/installing-trellis/>`_

Structure as recommended by Roots is

.. code::

     example.com/      # → Root folder for the project
     ├── trellis/      # → Your clone of this repository
     └── site/         # → A Bedrock-based WordPress site
         └── web/
             ├── app/  # → WordPress content directory (themes, plugins, etc.)
             └── wp/   # → WordPress core (don't touch!)

Set up a directory for your project:

.. code-block:: console

     [user@localhost]$ mkdir example.com && cd example.com


Then clone the repository:

.. code-block:: console

     [user@localhost]$ git clone --depth=1 git@github.com:roots/trellis.git && rm -rf trellis/.git


Clone Bedrock:

.. code-block:: console

    [user@localhost]$ git clone --depth=1 git@github.com:roots/bedrock.git site && rm -rf site/.git

Install the Galaxy Ansible Roles

.. code-block:: console

    [user@localhost]$ cd trellis && ansible-galaxy install -r requirements.yml



Next you need to change the wordpress_sites.yml. File for local
development is to be found at trellis/group_vars/development/wordpress_sites.yml


.. code::

    # group_vars/development/wordpress_sites.yml
    wordpress_sites:
      example.com:
        site_hosts:
          - example.dev
        local_path: ../site # path targeting local Bedrock site directory
        (relative to Ansible root)
        admin_email: admin@example.dev
        multisite:
          enabled: false
        ssl:
          enabled: false
        cache:
          enabled: false

You also need to edit vault.yml for local development:

.. code::

    #  group_vars/development/vault.yml
        vault_wordpress_sites:
          example.com:
            admin_password: admin
            env:
              db_password: example_dbpassword

This file contains all the database data.

Local Setup
~~~~~~~~~~~

How you install things locally depends partly on your operating system:

* OSX,
* Linux,
* Windows

and is not really part of this tutorial as we focus on the
DreamCompute part of things. I recommend following the Trellis
documentation on the `local development setup
<https://roots.io/trellis/docs/local-development-setup/>`_
. Just remember the earlier mentioned prerequisites. Without those on your
local server or PC you won't be able to get things started. This and the
proper changes in the Trellis configuration files for setting up a site
locally and remotely the way you want. See documentation on this at `Roots
Trellis Docs WordPress Sites
<https://roots.io/trellis/docs/wordpress-sites/>`_ .

Just make sure you have checked the following items:

* Configure your site(s) based on the WordPress Sites docs.
* Read the `development specific ones <https://roots.io/trellis/docs/wordpress-sites/#development>`_
* Make sure you've edited both group_vars/development/wordpress_sites.yml and
* edited group_vars/development/vault.yml.

They were discussed under installation earlier!

Then run the vagrant command from the command line. Do this inside the
trellis folder where the Vagrant File is:

.. code-block:: console

    [user@localhost]$ vagrant up


Bedrock
*******

For working with Bedrock - a Modern WordPress Stack - which is really
recommended we recommend you checking out the `Bedrock documentation
<https://roots.io/bedrock/>`_ . Just great to have a WordPress Stack with a
logical file structure, dependency management with Composer, easy WordPress
configuration and enhanced security!

Setting Up Your Instance
~~~~~~~~~~~~~~~~~~~~~~~~

Go to your DreamCompute Dashboard and pick Ubuntu from the list under
images. This Trellis server setup on a DreamCompute instance is best done
using a Ubuntu 14.0.4 image on DreamCompute. You can also use a more
recent version of Ubuntu, Ubuntu 16.0.4. However, you will then be
forced to install an older version of Python - 2.x - side by side
with Python 3 on your DreamCompute instance. This you can do using:

.. code-block:: console

    [user@server]$ sudo apt-get install python

If you do not mind this extra step then do go ahead. Always nice to run a
more recent Ubuntu version, isn't it?

Just make sure you use Ubuntu as the Ansible playbooks used by Trellis to
run the LEMP setup are built with Ubuntu/Debian in mind.
Whichever Ubuntu version you pick, remember it's better to boot volume
backed instances as they are permanent as opposed to ephemeral disks.

Provisioning Your DreamCompute Instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Provisioning Trellis means setting up the actual LEMP stack for your
staging or production website. Staging and Production do not differ much.
Do remember you need a separate instance for both though!
**NB** Provisioning you normally do once you have worked out the proper
site setup and have setup things locally.

Passwordless SSH
****************

Trellis works best with passwordless SSH login so do make sure you have
added your public SSH key in the DreamCompute Dashboard.

DreamCompute allows you to add your key in the DC Dashboard under Access &
Security > Key Pairs.

Also make sure the file trellis/group_vars/all/users.yml has the proper
path to you SSH key on your box and that that is the one you added to the
DreamCompute Dashboard.

Configuration Files
~~~~~~~~~~~~~~~~~~~

Let's say you work locally and on production only and have worked out
things locally. Then you only need to set up / edit a couple of files for
provisioning your remote server - `setting up remote server documentation
<https://roots.io/trellis/docs/remote-server-setup/>`_
**NB** You can most of the time just copy data from the local development files.

WordPress Sites
***************

The first one to begin with is wordpress_sites.yml. This file is located
under trellis/group_vars/production. In this file you need to change the
following:

* name of site
* site_hosts
* repository (Github repository for example)
* multisite or not
* ssl or not and which provider
* cache enabled or not

This is basically the same for setting things up locally so if you did that
well, this should work out just fine.

Vault
*****

Once that is done you also need to edit vault.yml - extended documentation
at https://roots.io/trellis/docs/vault/ under trellis/group_vars/production
. There you have to add:


* vault_mysql_root_password
* vault_wordpress_sites (same as in wordpress_sites.yml)
* db_password
* auth_key
* secure_auth_key
* logged_in_key
* nonce_key
* auth_salt
* secure_auth_salt
* logged_in_salt
* nonce_salt

Generate your keys at the Roots `salts generator
<https://roots.io/salts.html>`_ .


Hosts
*****

Now under the trellis folder open hosts/production. That is a file where
you add your host details for making the real connection. If you do forget
it you will net be able to connect and sometimes not get any errors at all
. Here is an example:

.. code::

    # Add each host to the [production] group and to a "type" group such as
     [web] or [db].
    # List each machine only once per [group], even if it will host
    multiple sites.

    [production]
    example.com

    [web]
    example.com

You can either add the domain connected to the DreamCompute public IP
address using an A record or use the IP address itself. Better connect the
domain to your instance before you provision. See this `DreamHost KB
article on Custom DNS Records
<https://goo.gl/vYHa1h>`_ .

Users
*****

Wait, we skipped one more important file to attend to located in
trellis/group_vars/all. That is users.yml. DreamCompute does not work with
root but with the user ubuntu and that should be reflected in this file:


.. code::

    # Documentation: https://roots.io/trellis/docs/ssh-keys/
    admin_user: ubuntu
    # Also define 'vault_sudoer_passwords' (`group_vars/staging/vault.yml`,
     `group_vars/production/vault.yml`)
    users:
      - name: "{{ web_user }}"
        groups:
          - "{{ web_group }}"
        keys:
          - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
          # - https://github.com/username.keys
      - name: "{{ admin_user }}"
        groups:
          - sudo
        keys:
          - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
          # - https://github.com/username.keys
    web_user: web
    web_group: www-data
    web_sudoers:
      - "/usr/sbin/service php7.0-fpm *"

Everything else in this file can stay the same. Do notice where it is
grabbing the SSH keys from. If you have keys with a different name or
located elsewhere you do need to change those lines as well.

Push to Remote DreamCompute Instance
************************************

Double check you have done the following:

* Configure your WordPress sites in group_vars/<environment>/wordpress_sites.yml
* configure all in in group_vars/<environment>/vault.yml (see the Vault docs for how to encrypt files containing passwords)
* Add your server IP/hostnames to hosts/<environment>
* Specify public SSH keys for users in group_vars/all/users.yml (see the SSH Keys docs)

When all that is good you can go ahead and push to the remote server using:

.. code-block:: console

    [user@localhost]$ ansible-playbook server.yml -e env=<environment>

Here *environment* will be production if you are pushing to production.
Staging is the other option.

**Note** Please understand that provisioning will take quite some time as
a full stack server will be installed with Nginx, MariaDB, PHP 7 and
beautiful things such as SSL, HTTP2 and so on. Also it takes care of
setting up WordPress on the server. All in all a pretty great feat.


Deploying your site to DreamCompute
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You have to realize that provisioning is just setting up your server for
working with WordPress really well and at lightning speed. The instance is
still not loading a site at all and going to the IP address or domain will
show you a nice Nginx 404 as nothing can be found. You simply need to
push your locally deployed WordPress site to the server still. Once that
is done you still either have to go through the installation process or
import and existing database.

For deploys, there are a couple more settings needed besides the ones you
did for provisioning:

* repository (required) - git URL of your Bedrock-based WordPress project (in SSH format: git@github.com:roots/bedrock.git)
* repo_subtree_path (optional) - relative path to your Bedrock/WP directory in your repository if its not the root (like site in roots-example-project)
* branch (optional) - the git branch to deploy (default: master)

You can deploy with a single command:

.. code-block:: console

    [user@localhost]$ ./deploy.sh <environment> <domain>

where the environment can again be staging or production .

**NOTE**
Make sure you have SSH Agent forwarding set up properly. Read more on it
at the `Using SSH Agent Forwarding
<https://developer.github.com/guides/using-ssh-agent-forwarding/>`_ article
at Github.

Issues setting up Trellis
~~~~~~~~~~~~~~~~~~~~~~~~~

If you do run into issues ask a question at `Roots Discourse
<https://discourse.roots.io/c/trellis>`_
This is the dedicated forum sub section for Trellis and that is where you
can find the experts you need debugging issues. Many errors with possible
solution can also be found at the Imagewize Blog article called `Roots
Trellis Errors
<https://imagewize.com/web-development/roots-trellis-errors/>`_ .

.. meta::
    :labels: trellis wordpress nginx
