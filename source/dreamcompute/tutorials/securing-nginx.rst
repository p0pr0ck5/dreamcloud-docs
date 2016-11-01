====================================================================
The most important steps to take to make an Nginx server more secure
====================================================================

Nginx is a high performance web server designed for serving high-performance,
scalable applications in an efficient, responsive manner. It can be used to
serve static content, load balance HTTP requests, and reverse proxy FCGI/PSGI/
USWGI, and arbitrary TCP connections. Given this, it's important to be able
to securely configure and deploy Nginx installations to provide a secure
web frontend for your application and minimize attack surfaces.

Securing the Binary
~~~~~~~~~~~~~~~~~~~

Keep Updated
------------

Nginx's core codebase (memory management, socket handling, etc) is very secure
and stable, though vulnerabilities in the main binary itself do pop up from time
to time. For this reason it's very important to keep Nginx up-to-date. Most
modern Linux distros will not push the latest version of Nginx into their
default package lists, so to install the latest version of Nginx via a package,
you may need to add additional package repositories to your system. See
`Nginx's documentation <http://nginx.org/en/linux_packages.html#stable>`_ for
per-distro details.

Compiling from Source
---------------------

As an alternative to building packages, it's possible (and easy!) to build Nginx
from source. Doing so allows you to run the latest version available from the
Nginx development team, and allows for additional security configurations (as
we'll see in a bit). Building from source requires a few steps. First, we need
the source archive, which we'll download from the office Nginx site. We'll use
the current stable branch as of this writing, 1.10.1:

.. code-block:: console

    # grab the latest tarball from the official Nginx website
    # we will use the latest stable, not mainline, version
    [user@server]$ wget http://nginx.org/download/nginx-1.10.1.tar.gz

We'll also want to grab the developer's signing key and verify the contents of
our download. First, we'll need the signing key, which we can download from
a public PGP keyserver:

.. code-block:: console

    [user@server]$ gpg --keyserver pgp.mit.edu --recv a1c052f8
    gpg: requesting key A1C052F8 from hkp server pgp.mit.edu
    gpg: key A1C052F8: public key "Maxim Dounin <mdounin@mdounin.ru>" imported
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   3  signed:   5  trust: 0-, 0q, 0n, 0m, 0f, 3u
    gpg: depth: 1  valid:   5  signed:   0  trust: 4-, 0q, 0n, 0m, 1f, 0u
    gpg: next trustdb check due at 2017-11-22
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)

Next, we'll grab the signature for this tarball:

.. code-block:: console

    [user@server]$ wget http://nginx.org/download/nginx-1.10.1.tar.gz.asc

And finally, we'll verify the signature:

.. code-block:: console

    [user@server]$ gpg --verify nginx-1.10.1.tar.gz.asc nginx-1.10.1.tar.gz
    gpg: Signature made Tue 31 May 2016 06:58:32 AM PDT using RSA key ID A1C052F8
    gpg: Good signature from "Maxim Dounin <mdounin@mdounin.ru>"
    Primary key fingerprint: B0F4 2533 73F8 F6F5 10D4  2178 520A 9993 A1C0 52F8

From here, we will unpack the tarball, compile Nginx, and install it:

.. code-block:: console

    [user@server]$ tar -zxf nginx-1.10.1.tar.gz
    [user@server]$ cd nginx-1.10.1/
    [user@server]$ ls
        auto  CHANGES  CHANGES.ru  conf  configure
        contrib  html  LICENSE  man README  src

    [user@server]$ ./configure && make && make install

Removing Unnecessary Modules
----------------------------

By default, Nginx compiles with a number of modules that extend its
functionality. These allow Nginx to be extended to perform a number of functions
but it's unlikely that every module will be used on any given server. It's
recommended to remove unused modules to reduce the size of the compiled binary,
and reduce the attack surface the Nginx presents to the world (for example, a
vulnerability found in the uwsgi proxy would not be exploitable against a
server that does not leverage the uswgi module). Removing modules can be done at
compile-time via the configure script. For example:

.. code-block:: console

    # disable the ngx_http_uwsgi_module
    [user@server]$ ./configure --without-http_uwsgi_module

The configure script provided with the Nginx script provides a large number of
compile-time options.


Securing Configurations
~~~~~~~~~~~~~~~~~~~~~~~

Computers are only as smart as the people using them. Nginx is built to be
stable and secure, but it will only be a secure as the user who configures it.
Once Nginx is built and installed, configure the server to be as minimal as
possible is important.

Run as an Unprivileged User
---------------------------

In security, the principle of least privilege states that an entity should be
given no more permission than necessary to accomplish its goals within a given
system. In the context of our web server, this means locking down Nginx to run
only with the permissions necessary to run. A first step in this process is to
configure Nginx to run as an unprivileged system user (e.g., not root). This is
done via the `user` directive in the Nginx configuration file:

.. code::

    # configure a non-privileged user. this user must exist on your system
    user nginx;


Disable Server Tokens
---------------------

The HTTP spec recommends (but not requires) that web servers identify themselves
via the `Server` header. Historically, web servers have included their version
information as part of this header. Disclosing the version of Nginx running can
be undesirable, particularly in environments sensitive to information disclosure.
Nginx can be configured to not display its version in the `Server` header:

.. code::

    server_tokens off;

Hide Upstream Proxy Headers
---------------------------

In the same vein, when Nginx is used to proxy requests from an upstream server
(such as a PHP-FPM instance), it can be beneficial to hide certain headers sent
in the upstream response (for example, the version of PHP running). For example,
consider the following response from an Nginx server running a PHP application:

.. code-block:: console

    [user@server]$ curl -I http://example.com
    HTTP/1.1 200 OK
    Server: nginx
    Content-Type: text/html; charset=UTF-8
    Connection: keep-alive
    Vary: Accept-Encoding
    X-Powered-By: PHP/5.3.3

Disclosing the version of PHP can be undesirable; Nginx configurations make this
easy to hide with the `proxy_hide_header` directive:

.. code::

    proxy_hide_header X-Powered-By;

Our request to the same server would now look like:

.. code-block:: console

    [user@server]$ curl -I http://example.com
    HTTP/1.1 200 OK
    Server: nginx
    Content-Type: text/html; charset=UTF-8
    Connection: keep-alive
    Vary: Accept-Encoding

Add Security Headers
--------------------

In addition to masking sensitive information, Nginx can be used to inject
headers with security-positive implications into responses as well. For
example, adding and `X-Frame-Options` header to prevent clickjacking attacks
is trivial to do:

.. code::

    add_header X-Frame-Options SAMEORIGIN;

This directive can also be used to add arbitrary headers at your whim.

Restrict Access by IP
---------------------

Sensitive areas of websites, such as admin control panels, should have strict
access controls placed on them. Nginx makes it easy to whitelist IP access to
certain locations of your website and deny traffic to all other IP addresses:

.. code::

    location /wp-admin {

        # allow access from one IP and an additional IP range,
        # and block everything else
        allow 1.2.3.4;
        allow 192.168.0.0/24;
        deny all;
    }

Restrict Access by Password
---------------------------

Access to certain locations can also be set via password-based credentials,
using the same format that Apache's .htaccess and .htpasswd files use:

.. code::

    location /wp-admin {
        auth_basic "Admin Area";
        auth_basic_user_file /path/to/htpasswd;
    }

Where the contents of `path/to/htpasswd` looks something like:

.. code::

    user1:password1
    user2:password2
    user3:password3

Securing SSL/TLS
~~~~~~~~~~~~~~~~

Nginx excels at serving SSL/TLS traffic. Configuring your web server to provide
securing SSL/TLS configurations for clients is essential to maintaining a secure
connection.

As a note, it's strongly recommended that encrypted traffic use only newer TLS
protocols, instead of legacy SSL. Both versions of SSL widely available today
(SSLv2 and SSLv3) have severe security flaws, and should never be used in
productions environments. Historically, the configurations associated with
SSL/TLS configuration in Nginx are prefixed with `ssl`; to promote the use of
modern security protocols, we will use the term 'TLS' when referencing encrypted
(HTTPS) traffic, and 'ssl' when applicable to specific Nginx configuration
directives.

Turn TLS On
-----------

It goes without saying, but in order to serve encrypted traffic, SSL/TLS needs
to be enabled for your server. Fortunately, encrypted connections can be
enabled/disabled on a per-server basis in Nginx:

.. code::

    server {
        # regular server listening for HTTP traffic
        listen 80;
    }

    server {
        # server listening for SSL traffic on port 443;
        listen 443 ssl;
    }

Enable Strong TLS Ciphers
-------------------------

By default, Nginx allows for a wide variety of cryptographic ciphers to be used
in TLS connections. Some of these ciphers are legacy offerings that are weak or
prone to attack, and shouldn't be used. We recommend using the Modern or
Intermediate cipher suites outlined by Mozilla (the modern list of ciphers is
stronger, but will cause connectivity problems for older platforms like Internet
Explorer or Windows XP). Additionally, it's recommended that the server prefer
which cipher to be used:

.. code::

    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;

Enable TLS Session Caching
--------------------------

Opening a new TLS connection to a server is very expensive as a result of the
cryptographic protocols involved. To maintain a high-performance environment,
it's recommended to cache existing TLS connections so that each new request from
a client/browser does not need to perform the full TLS handshake:

.. code::

    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 5m;

Use Custom Diffie-Hellman Parameters
------------------------------------

The Logjam attack, published in 2015, showed that it was possible for attackers
(such as nation-state actors) to break the Diffie-Hellman key exchange, used to
implement forward secrecy (essentially, another layer on top of existing
encrypted messages). Mitigating this attack is possible in Nginx by computing a
unique set of Diffie-Hellman parameters and configuring Nginx to use this value:

.. code-block:: console

    # build a 2048-bit DH prime
    [user@server]$ openssl dhparam 2048 > /path/to/dhparam

From here we only need to tell Nginx to use our custom values:

.. code::

    ssl_dhparam /path/to/dhparam;

For more information on the Logjam attack, see https://weakdh.org/

Force All Connections over TLS
------------------------------

Encrypted communications are only useful when actually in use. If desirable, it
is possible to tell browsers to only use TLS connections for your site. This
is accomplished with the `Strict-Transport-Security` header, which can be added
in your Nginx config as we've seen before:

.. code::

    add_header Strict-Transport-Security max-age=15768000;

We can also configure Nginx to send a 301 redirect for plaintext HTTP requests
to the TLS version of your site:

.. code::

    server {
        listen 80;
        server_name example.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name example.com;

        # the rest of the appropriate server block below...
    }

Additional Security Measures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Beyond the basics of installing a secure Nginx binary, locking down access to
sensitive areas of your site, and properly serving TLS connections, there are
some additional steps that can be taken for the extra security-conscious user.

Install a WAF
-------------

A WAF (web application firewall) is a piece of software designed to inspect
HTTP/HTTPS traffic, deny malicious requests, and generally act as an additional
layer of security in your web stack. A properly configured WAF can protect
your site from SQLi, XSS, CSRF, and DDoS attacks, as well as provide brute force
attack mitigation and zero-day threat patching. There are a few open-source WAF
options available for Nginx:

* `ModSecurity <https://www.modsecurity.org/>`_, originally written as a WAF for
  Apache servers, is the de-facto standard for open-source WAF solutions. Recent
  work on the project has shifted focus toward Nginx support; see the project's
  `GitHub page <https://github.com/SpiderLabs/ModSecurity>`_
  for more detail on installation and configuration.

* `Naxsi <https://github.com/nbs-system/naxsi>`_ is a lightweight alternative to
  ModSecurity, designed as a native Nginx module, and focuses on XSS/SQLi
  prevention in request parameters.

* For users of the OpenResty bundle seeking a scriptable, high-performance WAF,
  check out `lua-resty-waf <https://github.com/p0pr0ck5/lua-resty-waf>`_, which
  seeks to provide a ModSecurity- compatible rule engine integrated into the
  Nginx + LuaJIT ecosystem.

Automated Log Analysis + Monitoring
-----------------------------------

Programs like Fail2Ban can be used to monitor Nginx access and error logs,
searching for attack patterns and taking actions against the attacking client
(such as dropping IP addresses, reporting malicious behavior to the IP's owner,
etc). Fail2Ban is extensible, allowing you to write your own search pattern and
response behavior.

Limit Input Traffic via IPTables
--------------------------------

Beyond securing Nginx itself, it's important to secure the host environment used
to host your web server. Locking down access to things like SSH can greatly
increase the security of the host by preventing intrusion attempts. A common
approach is to whitelist known IPs that will access your host via SSH, and deny
all other port 22 traffic, or to use a jump box that strictly filters shell
access.

.. meta::
    :labels: nginx security
