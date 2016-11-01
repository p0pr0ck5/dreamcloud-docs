=====================================================================
The most important steps to take to make an Apache server more secure
=====================================================================

Apache is the most popular open source web server available for modern Linux
servers. It offers flexible configuration allowing for a wide variety of uses,
from serving basic HTML sites, to complex PHP/Passenger applications, to
proxying requests as a reverse proxy gateway. Given its popularity and easy of
use, it's essential to install and maintain a secure environment for Apache
installations.

Keep Apache Updated
~~~~~~~~~~~~~~~~~~~

Apache has a good security track record, and security bugs are seldom found
within the web server itself. Still, it's important to keep Apache updated in
order to take advantage of the latest security, stability and features
available. Generally, this is simply a matter of keeping the Apache package
provided by the distro's OS updated (e.g. via `apt`, `yum`, etc). It's also
recommended that Apache server operators following the
`Apache Server Announcements <http://httpd.apache.org/lists.html>`_ mailing list
to stay abreast of the latest news from the Apache development team.

Securing Configurations
~~~~~~~~~~~~~~~~~~~~~~~

Computers are only as smart as the people using them. Apache is built to be
stable and secure, but it will only be a secure as the user who configures it.
Once Apache is built and installed, it's important to configure the server to be
as minimal as possible.

Run as an Unprivileged User
---------------------------

In security, the principle of least privilege states that an entity should be
given no more permission than necessary to accomplish its goals within a given
system. In the context of our web server, this means locking down Apache to run
only with the permissions necessary to run. A first step in this process is to
configure Apache to run as an unprivileged system user (e.g., not root). This is
done via the `User` and `Group`  directives in the Apache configuration file:

.. code::

    # configure a non-privileged user. this user must exist on your system
    User apache;
    Group apache;

Apache servers distributed as a common OS package may also use a user and group
name such as `www-data` or `nobody`. Regardless of the choice of user's name,
make sure that the user/group selection has as few rights as necessary to run
properly.

Disable Server Tokens
---------------------

The HTTP spec recommends (but not requires) that web servers identify themselves
via the `Server` header. Historically, web servers have included their version
information as part of this header. Disclosing the version of Apache running can
be undesirable, particularly in environments sensitive to information
disclosure. Configure Apache not to display its version in `Server` header:

.. code::

    ServerTokens ProductOnly;

Disable .htaccess Files
-----------------------

`.htaccess` files are a powerful feature that allow Apache to have its
configuration extended outside its main config file. While this may be
convenient, it does present a security risk, as Apache will read any
.htaccess file in its path- even ones placed by an attacker that could
compromise the server. It may be desirable to lock down configuration by
disabling .htaccess files entirely, via the `AllowOverride` directive:

.. code::

    AllowOverride None

Additionally, fine-grained control of which Apache directives can be used in
.htaccess files can also be controlled by `AllowOverride`:

.. code::

    AllowOverride AuthConfig Indexes

In the example above, all directives that are neither in the group AuthConfig
nor Indexes cause an internal server error.

Restrict Access by IP
---------------------

Sensitive areas of websites, such as admin control panels, should have strict
access controls placed on them. Access makes it easy to whitelist IP access to
certain locations of your website and deny traffic to all other IP addresses:

.. code-block:: apacheconf

    <Directory "/wp-admin">

        # allow access from one IP and an additional IP range,
        # and block everything else
        Order Allow,Deny
        Allow from 1.2.3.4
        Allow from 192.168.0.0/24
    </Directory>

In this example, the use of the `Order` directive instructs Apache to allow
requests coming from IP addresses listed in the `Allow` blocks, and to deny all
other traffic.

Restrict Access by Password
---------------------------

Access to certain locations can also be set via password-based credentials,
using the `htpasswd` utility. First, create a file called  `.htpasswd` to store
the desired user/password combinations:

.. code-block:: console

    [root@server]# htpasswd -c /path/to/.htpasswd user1

The `htpasswd` command creates the file `/path/to/.htpasswd` if it doesn't
exist, and prompt for a password. To add another user, simply run the command,
leaving out the `-c` argument:

.. code-block:: console

    [root@server]# htpasswd /path/to/.htpasswd user2

Once you've created the user, configure Apache to read the password file and
control access to the desired directory:

.. code-block:: apacheconf

    <Directory "/wp-admin">
        AuthType Basic
        AuthName "Restricted Content"
        AuthUserFile /path/to/.htpasswd
        Require valid-user
    </Directory>

Preventing DoS Attacks
~~~~~~~~~~~~~~~~~~~~~~

The default model in which Apache processes requests, called prefork mode, is
subject to an attack known as a Slowloris attack. A Slowloris attack is a form
of DoS (Denial of Service) attack in which the Apache server is forced to wait
on requests from malicious clients taking a long time to send traffic, thus
forcing legitimate requests to time out or be ignored entirely. Thankfully,
modern Apache servers are capable of mitigating this threat with a few
additional configuration directives.

Enable mod_reqtimeout
---------------------

`mod_reqtimeout` is an Apache module designed to shut down connections from
clients taking too long to send their request, such as is seen in a Slowloris
attack. This module provides a directive that allows Apache to close the
connection if it senses that the client is not sending data quickly enough. For
example:

.. code::

    RequestReadTimeout header=10-20,MinRate=500 body=20,MinRate=500

In this example, Apache will close the connection if the client takes more than
10 seconds to send its HTTP headers, or if the client takes more than 20 seconds
to send headers at a rate of 500 bytes per second. Apache will also close the
connection if the client takes more than 20 seconds to send its request body,
but will allow the request to continue as long as the client sends more than
500 bytes per second. This configuration allows clients will poor TCP connection
quality (such as remote clients with high latency, or those on low-grade
cellular or satellite networks) to send requests, while still protecting against
known fingerprints of the Slowloris attack. `RequestReadTimeout` configurations
can be complex; more information about this directive can be found at the module
`documentation page <https://httpd.apache.org/docs/2.4/mod/mod_reqtimeout.html>`_.

Securing SSL/TLS
~~~~~~~~~~~~~~~~

Apache excels at serving SSL/TLS traffic. Configuring a web server to provide
secure SSL/TLS configurations for clients is essential to maintaining a secure
connection.

As a note, it's strongly recommended that encrypted traffic use only newer TLS
protocols, instead of legacy SSL. Both versions of SSL widely available today
(SSLv2 and SSLv3) have severe security flaws, and should never be used in
productions environments. Historically, the configurations associated with
SSL/TLS configuration in Apache are prefixed with `SSL`; to promote the use of
modern security protocols, in this tutorial the term 'TLS' is used when
referencing encrypted (HTTPS) traffic, and 'ssl' when applicable to specific
Apache configuration directives.

Turn TLS On
-----------

In order to serve encrypted traffic, SSL/TLS needs to be enabled in Apache.
Enable secure communications with the `SSLEngine` directive:

.. code-block:: apacheconf

    <VirtualHost 192.168.1.1:443>
        SSLEngine on
        SSLCertificateFile /path/to/cert
        SSLCertificateKeyFile /path/to/key
    </VirtualHost>

Enable Strong TLS Ciphers
-------------------------

By default, Apache allows for a wide variety of cryptographic ciphers to be used
in TLS connections. Some of these ciphers are legacy offerings that are weak or
prone to attack, and shouldn't be used. Dreamhost recommends using the Modern or
Intermediate cipher suites outlined by Mozilla (the modern list of ciphers is
stronger, but will cause connectivity problems for older platforms like Internet
Explorer or Windows XP). Additionally, it's recommended that the server prefer
which cipher to be used:

.. code::

    SSLCipherSuite 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    SSLHonorCipherOrder on
    SSLProtocol all -SSLv2 -SSLv3

Enable TLS Session Caching
--------------------------

Opening a new TLS connection to a server is very expensive as a result of the
cryptographic protocols involved. To maintain a high-performance environment,
cache existing TLS connections so that each new request from a client/browser
does not need to perform the full TLS handshake:

.. code::

    SSLSessionCache shm:/path/to/session_cache(512000)
    SSLSessionCacheTimeout 300

Use Custom Diffie-Hellman Parameters
------------------------------------

The Logjam attack, published in 2015, showed that it was possible for attackers
(such as nation-state actors) to break the Diffie-Hellman key exchange, used to
implement forward secrecy (essentially, another layer on top of existing
encrypted messages). Mitigating this attack is possible in Apache by computing a
unique set of Diffie-Hellman parameters and configuring Apache to use this value:

.. code-block:: console

    # build a 2048-bit DH prime
    [user@server]$ openssl dhparam 2048 > /path/to/dhparam

From here, add the params to the end of the file noted in the
`SSLCertificateFile` directive:

.. code-block:: console

    [root@server]# cat /path/to/custom/dhparam >> /path/to/sslcertfile

For more information on the Logjam attack, see https://weakdh.org/

Force All Connections over TLS
------------------------------

Encrypted communications are only useful when actually in use. Apache can tell
browsers to only use TLS connections for your site. This is accomplished with
the `Strict-Transport-Security` header:

.. code::

    Header always set Strict-Transport-Security max-age=15768000;

For all plaintext connections, configure Apache to send a 301 redirect for
requests to the TLS version of the site:

.. code-block:: apacheconf

    <VirtualHost 192.168.1.1:80>
        [...]
        ServerName example.com
        Redirect permanent / https://example.com/
    </VirtualHost>

Additional Security Measures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Beyond the basics of installing a secure Apache binary, locking down access to
sensitive areas of your site, and properly serving TLS connections, there are
some additional steps that the extra security-conscious user can take:

Install a WAF
-------------

A WAF (web application firewall) is a piece of software designed to inspect
HTTP/HTTPS traffic, deny malicious requests, and generally act as an additional
layer of security in an HTTP web stack. A properly configured WAF can protect
your site from SQLi, XSS, CSRF, and DDoS attacks, as well as provide brute force
attack mitigation and zero-day threat patching. The most popular and stable
WAF for Apache is `ModSecurity <https://www.modsecurity.org/>`_; see the
project's `GitHub page <https://github.com/SpiderLabs/ModSecurity>`_
for more detail on installation and configuration.

Automated Log Analysis + Monitoring
-----------------------------------

Programs like Fail2Ban can be used to monitor Apache access and error logs,
searching for attack patterns and taking actions against the attacking client
(such as dropping IP addresses, reporting malicious behavior to the IP's owner,
etc). Fail2Ban is extensible, allowing for the creation of unique search
patterns and response behaviors.

Limit Input Traffic via IPTables
--------------------------------

Beyond securing Apache itself, it's important to secure the host environment
used to host the web server. Locking down access to things like SSH can greatly
increase the security of the host by preventing intrusion attempts. A common
approach is to whitelist known IPs that will access the host via SSH, and deny
all other port 22 traffic, or to use a jump box that strictly filters shell
access.

.. meta::
    :labels: apache security
