==================================================
Installing ModSecurity with Apache on Ubuntu 14.04
==================================================

ModSecurity is an open source web application firewall (WAF) designed as a
module for Apache web servers. ModSecurity provides a flexible rule engine,
allowing users to write (or use third-party) rules for protecting websites
from attacks such as XSS, SQLi, CSRF, DDoS, and brute force login (as well
as a number of other exploits). This tutorial walks through the basics
of installing and configuring ModSecurity for an Apache web server. This
tutorial assumes that Apache is already installed and running.

Installing ModSecurity
~~~~~~~~~~~~~~~~~~~~~~

Ensure that the system package sources are up to date:

.. code-block:: console

    [root@server]# apt-get update

Next, install ModSecurity:

.. code-block:: console

    [root@server]# apt-get install libapache2-mod-security2

This automatically installs and activates ModSecurity. In order to begin using
ModSecurity, a usable configuration file must be put into place. The ModSecurity
package provided for Ubuntu contains a default recommended config file that can
be used as a starting point:

.. code-block:: console

    [root@server]# mv /etc/modsecurity/modsecurity.conf-recommended \
        /etc/modsecurity/modsecurity.conf

Once this is in place, reload Apache for the default ModSecurity config file to
take effect:

.. code-block:: console

    [root@server]# service apache2 reload

Configuring ModSecurity
~~~~~~~~~~~~~~~~~~~~~~~

The recommended default config file provided for ModSecurity has very few
actual protective rules configured, but is a good starting point. In this
tutorial the OWASP Core Rule Set (CRS) is used to provide additional
protection.

Enabling CRS Rulesets
---------------------

The Ubuntu package for ModSecurity recommends a separate package containing the
CRS rulesets, which can be used as an extra source of rules for WAF.
Navigate to the directory containing these rules:

.. code-block:: console

    [root@server]# cd /usr/shared/modsecurity-crs

Configure ModSecurity to read rule files from the `activated_rules` directory.
Add the following directives to the
`/etc/apache2/mods-enabled/security2.conf` file:

.. code-block:: apacheconf

    IncludeOptional "/usr/share/modsecurity-crs/*.conf"
    IncludeOptional "/usr/share/modsecurity-crs/activated_rules/*.conf"

This instructs ModSecurity to attempt to use any files ending in `conf`
as configuration files. More information is available in the README file
in the `activated_rules` directory.

Once this is done, link the desired rulesets into the newly included locations.
For example, to add rules designed to protect against SQL injection
attacks, link in the `sql_injection_attacks` file:

.. code-block:: console

    [root@server]# cd /usr/share/modsecurity-crs/
    [root@server]# ln -s ./base_rules/modsecurity_crs_41_sql_injection_attacks.conf \
        ./activated_rules/

Of course, it's possible to link only certain rulesets, or entire groups,
depending on your needs. The CRS is also distributed with custom and
experimental rulesets to detect and mitigate a wide variety of emerging threats.
Rulesets for specific CMS/application installations, such as WordPress and
Joomla, are also available in the `slr_rules` directory (though as a free WAF
ruleset offering, these rulesets are not always current with the latest
threats).

Any time the ModSecurity configuration is adjusted, Apache must be reloaded
in order for the rules to take effect:

.. code-block:: console

    [root@server]# service apache2 reload

Activating ModSecurity
----------------------

ModSecurity initially runs in `DetectionOnly` mode, in which the WAF
examines HTTP(S) traffic, but not actually block malicious requests. This
must be adjusted in order for ModSecurity to deny attack traffic. In the
file `/etc/modsecurity/modsecurity.conf`, find the directive `SecRuleEngine`:

.. code::

    SecRuleEngine DetectionOnly

And set its value to `On`:

.. code::

    SecRuleEngine On

And of course, reload Apache to effect the changes:

.. code-block:: console

    [root@server]# service apache2 restart

Further Configuration
~~~~~~~~~~~~~~~~~~~~~

WAF environments can be complex and time-consuming to tune and adjust based on
your server's needs; this is largely why the CRS was created. If you need to
write or change custom rules, it's recommended to read though the `ModSecurity
reference manual <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual>`_.
Additionally, the `modsecurity-users` mailing list and `#modsecurity` room on
Freenode IRC are excellent resources for experienced ModSecurity users and
developers.

.. meta::
    :labels: apache security
