Setting up Let's Encrypt on DreamCompute with Nginx
===================================================

What is Let's Encrypt?
----------------------

`Let's Encrypt <https://letsencrypt.org>`__ is a new certificate
authority that provides absolutely free secure certificates to help
get to 100% HTTPS on the Internet. DreamHost has integrated Let's
Encrypt support into our panel for hosted services, but if you want
to set up automatically-renewing certificates for domains you host
on a DreamCompute instance, you'll need to do a little bit of manual
installation. But the good news is, it doesn't take long, and once
you finish the setup, you should never have to worry about renewing
a certificate ever again!


Get the code
------------

You'll need to SSH to your DreamCompute instance. It shouldn't matter
too much which distribution of Linux you're running, but make sure
you have the ``git`` package installed so that you can clone the
letsencrypt repository, like so:

.. code-block:: console

    [user@server]$ sudo -s
    [root@server]# cd /opt
    [root@server]# git clone git://github.com/letsencrypt/letsencrypt

Get your first certificate
--------------------------

Before you do this, you'll need to make sure that your domain is
actually pointing at your DreamCompute instance's IP address, and
that your webserver is configured to respond to requests for your
domain name. Let's Encrypt performs checks to make sure that you
control domain names that you request certificates for.

But, let's say that you have ``example.com`` configured with a DNS
``A`` record pointing at the IP address for your instance, and you
have ``nginx`` already configured properly to respond
to requests for ``example.com``. (Configuring your webserver is kind
of out of the scope of this guide, but there are `plenty of tutorials
<https://www.nginx.com/resources/wiki/start/>`__ out there.)

These sample snippets assume that the webserver is configured to
serve files for ``example.com`` from the location ``/srv/example.com``
on your instance. Make sure to update that location to match your
domain's document root!

If you're using ``nginx``, or ``lighttpd``, or any other webserver
that supports HTTPS, it's a good idea to use the ``certonly`` plugin:

.. code-block:: console

    [root@server]# cd /opt/letsencrypt
    [root@server]# ./letsencrypt-auto certonly --webroot --webroot-path /srv/example.com -d example.com

Either way, this will prompt you for some information including
your email address. Fill it in with valid information and you
should get a shiny new certificate! ``Nginx`` and other users will
need to update configurations
by hand to point at the new SSL certificate and key files. A sample
nginx snippet is included below (insert something like this into the
``server {`` stanza for your domain):

.. code:: nginx

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

Adding a subdomain to an existing certificate
---------------------------------------------

If you just realized that you also need a certificate for a subdomain,
don't worry! You can add a new subdomain to your existing cert at any
time, by simply calling ``letsencrypt-auto`` again like so...

.. code-block:: console

    [root@server]# cd /opt/letsencrypt
    [root@server]# ./letsencrypt-auto certonly --webroot --webroot-path /srv/example.com -d example.com --webroot-path /srv/sub.example.com -d sub.example.com

This is, of course, assuming that you have a different document root
for the files for your subdomain. You can omit the additional
``--webroot-path`` argument if the document root is the same for
the top-level domain and the subdomain. Always remember to specify the
``--webroot-path`` *before* each ``-d`` argument, because the ``-d``
argument uses the most-recently-specified ``webroot-path`` variable
supplied.

Automatic renewal
-----------------

Now, the best part about using Let's Encrypt (well, aside from the free
certificates): You can have your system automatically renew all of the
certificates for you. I wrote a small shell script I called
``/usr/local/bin/update_certs`` which looks like this:

.. code-block:: bash

    #!/bin/bash

    /opt/letsencrypt/letsencrypt-auto renew

    systemctl reload nginx.service

Using ``cron``, I have this scheduled like so:

.. code::

    30 0 * * 0 /usr/local/bin/update_certs

And now, my system attempts to renew all of my certificates once a week.
If there are no certificates in danger of expiring soon, nothing bad
happens, but if they would otherwise expire, then they get renewed and
I don't have to think about it.

.. meta::
    :labels: nginx https letsencrypt
