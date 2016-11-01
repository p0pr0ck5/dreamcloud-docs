=================================================
How to deploy software with Dokku on DreamCompute
=================================================

What is Dokku, and why would I want it?
---------------------------------------

Wouldn't it be nice to be able to ``git push`` directly from a
repository on your local machine to your production server and have all
of your software's dependencies automatically installed?

With Dokku on DreamCompute, it's pretty easy to do. And because Dokku
implements Heroku's `buildpack specification
<https://devcenter.heroku.com/articles/buildpacks>`__, you can easily
deploy software written in a variety of languages - for example, PHP,
Golang, NodeJS, Python, and Ruby (including Ruby on Rails).

For the purposes of this guide, I'm going to assume you already have a
DreamCompute instance launched running Debian 8 (Jessie).

Installing Dokku
----------------

Before installing anything, it's usually a good idea to make sure that
the rest of your system's packages are up to date. For all of the
commands in this section, ssh into your Debian instance, and run:

.. code-block:: console

    [user@server]$ sudo apt-get update && sudo apt-get dist-upgrade

Dokku's repository is hosted on HTTPS, so the first step to installing
it is:

.. code-block:: console

    [user@server]$ sudo apt-get install apt-transport-https

Now, we're going to import the keys for the Docker and Dokku
repositories (the version of Docker that Debian Jessie ships with is too
old for Dokku, so we're going to use the upstream Docker repository).

.. code-block:: console

    [user@server]$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 0x2C52609D # this is the Docker key
    [user@server]$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 0xD59097AB # this is the Dokku key

If you've already pointed a wildcard `DNS <215414867>`__ entry at this
machine, you can
enable Dokku's vhost support (which will get you easy-to-use project
URLs like ``project.example.com``). If you don't have a wildcard DNS
entry pointing at this IP address, you should set vhost\_enable to
``false``.

We're going to preconfigure dokku before we install it using debconf:

.. code-block:: console

    [user@server]$ echo 'dokku dokku/web_config boolean false' | sudo debconf-set-selections
    [user@server]$ echo 'dokku dokku/vhost_enable boolean true' | sudo debconf-set-selections
    [user@server]$ echo 'dokku dokku/hostname string [your-domain]' | sudo debconf-set-selections
    [user@server]$ echo 'dokku dokku/key_file string /home/admin/.ssh/id_rsa.pub' | sudo debconf-set-selections

You should modify the configuration to suit your own setup, and make
sure that you point at the right SSH public key file.

Once you have the keys installed and dokku pre-configured, it's time to
tell apt how to find the new repositories, and install dokku:

.. code-block:: console

    [user@server]$ echo 'deb https://apt.dockerproject.org/repo debian-jessie main' | sudo tee /etc/apt/sources.list.d/docker.list
    [user@server]$ echo 'deb https://packagecloud.io/dokku/dokku/ubuntu/ trusty main' | sudo tee /etc/apt/sources.list.d/dokku.list
    [user@server]$ sudo apt-get update && sudo apt-get install dokku

Setting up your local git project
---------------------------------

If you want to play with this and you don't already have a
buildpack-compatible project to deploy, you can clone this project I
wrote in nodejs on your local machine to test your setup:

.. code-block:: console

    [user@localhost]$ git clone git://github.com/clee/p90xcalgen

nodejs projects require a ``Procfile`` specifying how to run the server,
and a ``package.json`` file describing the dependencies.

The project I linked to above uses this ``Procfile``:

::

    web: node app.js

And this ``package.json``:

.. code-block:: json

    {
        "name": "p90xcalgen",
        "version": "0.3.9",
        "private": true,
        "dependencies": {
            "express": "4.x.x",
            "body-parser": "1.x.x",
            "errorhandler": "1.x.x",
            "jade": ">= 1.x.x",
            "ejs": ">= 2.x.x"
        },
        "engines": {
            "node":  ">= 5.7.1"
        }
    }

If you're using your own project, refer to the buildpack documentation
to figure out if you need to make any changes so that the buildpack
knows how to deploy it. Rails projects should be automatically detected,
for example, but each language has different requirements.

Deploying to Dokku
------------------

Dokku has `some helpful documentation
<http://dokku.viewdocs.io/dokku~v0.6.2/application-deployment/>`__ which goes
into much more detail (especially if your application requires additional
services, like MySQL/PostgreSQL/redis/etc), but here's the short version for
a simple app with no database requirements like the example provided above.

You'll need to tell dokku about your project before you can deploy it.
On your Debian instance:

.. code-block:: console

    [user@localhost]$ dokku apps:create [project_name]

On your local machine, in your project's source folder:

.. code-block:: console

    [user@localhost]$ git remote add dokku dokku@[dreamcompute-IP]:[project_name]
    [user@localhost]$ git push dokku master

And voila! Assuming that you have configured everything correctly, you
should now have a working deployed application.

.. meta::
    :labels: dokku
