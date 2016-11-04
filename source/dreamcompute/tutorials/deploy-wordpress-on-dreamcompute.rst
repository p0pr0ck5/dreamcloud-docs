======================================================
Step-by-step guide to deploy WordPress on DreamCompute
======================================================

Preparation
~~~~~~~~~~~~

In this tutorial we are going to install WordPress on a DreamCompute
instance, including both the application itself and the database it uses.
We'll install and configure all necessary components without making use of
automatic configuration management systems.

This example will use an Ubuntu virtual machine, but you can use whichever
flavor of linux you prefer. As long as you can install Apache, PHP, and
database software, WordPress will be able to run.

Whichever machine type you pick, remember it's better to boot volume backed
instances as they are permanent as opposed to ephemeral disks.

Installing LAMP
~~~~~~~~~~~~~~~

While you certainly can install everything on it's own, the LAMP stack for
Ubuntu is perfect for WordPress. If necessary, you can follow the directions
on `how to configure LAMP
<215879467-How-to-Configure-LAMP-on-DreamCompute-running-Debian-or-Ubuntu>`_
but the overview is as follows:

.. code-block:: console

    [user@server]$ sudo apt-get install lamp-server^

This is interactive, so you'll be asked 'are you sure?' in some places, and
in others it will want a password for SQL. Make up a secure password for SQL
and save it as you will need this later to set up SQL for WordPress.

After it runs, you'll want to add mod_rewrite so WordPress can make pretty
pages:

.. code-block:: console

    [user@server]$ sudo a2enmod rewrite

Finally restart apache:

.. code-block:: console

    [user@server]$ sudo service apache2 restart

At this point, your server will be accessible via its IP address.

Create a User
~~~~~~~~~~~~~

While it's not required to make a separate user ID for WordPress, it's strongly
recommended for security. Having a user who only has access to the one website
will limit the havoc caused should that account be hacked. This also limits the
danger caused by rogue plugins or themes. The server itself will be safe,
containing the damage just to that user account.

To do this, we make a folder for the website:

.. code-block:: console

    [user@server]$ sudo mkdir /var/www/example.com

Then give your user access to that directory. This is so that you can install
the WordPress site, later you will change ownership of that directory to
www-data so Apache can access it.

.. code-block:: console

    [user@server]$ sudo chown -R $USER /var/www/example.com/

Add SSH Access
~~~~~~~~~~~~~~

WordPress users often need SSH access in order to do extra configuration with
WordPress. By default, this is disabled, so you will need to edit your config
using a text editor, such as vi, emacs, or nano (this one is easiest for
beginners).

.. code-block:: console

    [user@server]$ sudo vi /etc/ssh/sshd_config

Look for the setting of PasswordAuthentication, change it to "yes", and save
your file. Remember to restart SSHD once you've done this.

.. code-block:: console

    [user@server]$ sudo service sshd restart

Will this make your server less secure? Not significantly. As this new account
only has access to itself, it can only hack itself.

Add Your Domain
~~~~~~~~~~~~~~~

There are a few steps to set up your domain. First you'll need to `Setup DNS
for DreamCompute <218672058>`_ for all your domains.

Next you'll want to configure VirtualHosts so your server knows how to handle
the domain.

To do this, you need to make a .conf file:

.. code-block:: console

    [user@server]$ sudo touch /etc/apache2/sites-available/example.com.conf

It's recommended you name the file after your domain, so you can always know
what file is for what domain.

Edit that file and put this in:

.. code-block:: apacheconf

    <VirtualHost *:80>
        ServerName example.com
        ServerAdmin admin@example.com
        DocumentRoot /var/www/example.com
        <Directory /var/www/example.com>
                AllowOverride all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
        CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined
    </VirtualHost>

Once the site is added, we'll need to enable it via a command called a2ensite
(if you want to disable, it's a2dissite):

.. code-block:: console

    [user@server]$ sudo a2ensite

This will prompt you to pick what site you want to enable. Type it in, hit
enter, and you'll be told what's next.

.. code::

    Your choices are: 000-default default-ssl example.com
    Which site(s) do you want to enable (wildcards ok)?
    example.com
    Enabling site example.com.
    To activate the new configuration, you need to run:
      service apache2 reload

Remember this command. It's a fast way to enable sites without having to rename
or mess with files. Finally bounce your apache service so it reads the changes:

.. code-block:: console

    [user@server]$ sudo service apache2 reload

Create the Database and Users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

WordPress absolutely requires a database. You'll want to create one

.. code-block:: console

    [user@server]$ mysql -u root -p

Remember the password we set earlier? That's what it's for.

Your command prompt will be `mysql>` so let's make the database:

.. code-block:: sql

    mysql> CREATE DATABASE examplecom_wordpress;
    mysql> GRANT ALL ON examplecom_wordpress.* TO examplecom@localhost IDENTIFIED by 'PASSWORD';

Remember to change PASSWORD to an actually secure password.

You can check this by running the following command:

.. code-block:: console

    [user@server]$ mysql -u examplecom -p examplecom_wordpress

Install WP-CLI
~~~~~~~~~~~~~~

While this is optional, we strongly recommend this. DreamHost includes `WP-CLI
<http://wp-cli.org/>`_ on all servers due to its usefulness. To install, log
in as your default user (not the web user we created earlier) and run the
following:

.. code-block:: console

    [user@server]$ cd ~
    [user@server]$ curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

Check that it works:

.. code-block:: console

    [user@server]$ php wp-cli.phar --info

And if it does move it so everyone can use it!

.. code-block:: console

    [user@server]$ chmod +x wp-cli.phar
    [user@server]$ sudo mv wp-cli.phar /usr/local/bin/wp

That will make it accessible for all users.

Install WordPress
~~~~~~~~~~~~~~~~~

If you've installed WP-CLI, then all you have to do is this:

.. code-block:: console

    [user@server]$ wp core download

If you go to http://example.com now you'll get that 5 minute install page.

Of course since you have wp-cli you can also do this:

.. code-block:: console

    [user@server]$ wp core config --dbname=examplecom_wordpress --dbuser=examplecom --dbpass=PASSWORD
    [user@server]$ wp core install --url=http://example.com --title=DreamComputePress --admin_user=YOURUSERNAME --admin_password=PASSWORD --admin_email=admin@example.com --skip-email

If you use secure passwords like cWG8j8FPPj{T9UDL_PW8 then you MUST put quotes
around the password.

I chose to skip-emails since I'm making it right there.

Finally you must change ownership of the WordPress site to www-data and the
group to www-data as well so that the Apache webserver can access the files:

.. code-block:: console

    [user@server]$ chown -R www-data:www-data /var/www/example.com

Miscellaneous Stuff
~~~~~~~~~~~~~~~~~~~

The following will make WordPress run even better, but aren't required.

Make sure apt has the latest and greatest.

.. code-block:: console

    [user@server]$ sudo apt-get -y update

Make PHP Better
~~~~~~~~~~~~~~~

If you use a lot of media, install these to improve how PHP processes images.

.. code-block:: console

    [user@server]$ sudo apt install php-imagick php7.0-gd

Run a restart of apache when you're done:

Troubleshooting
~~~~~~~~~~~~~~~

If WordPress can't save files, you probably forgot to change ownership of the
WordPress site to www-data.

.. code-block:: console

    [user@server]$ sudo chown -R www-data:www-data /var/www/example.com/

If that still doesn't work, try this:

.. code-block:: console

    [user@server]$ sudo chgrp -R www-data /var/www/example.com/
    [user@server]$ sudo chmod -R g+w /var/www/example.com/

If pretty permalinks don't work, make sure you installed rewrite:

.. code-block:: console

    [user@server]$ sudo a2enmod rewrite && sudo service apache2 restart

And make absolutely sure you have AllowOverride set to All in your Virtual Host:

.. code-block:: apacheconf

    <Directory /var/www/example.com>
        AllowOverride all
    </Directory>

It won't work without it.

.. meta::
    :labels: wordpress
