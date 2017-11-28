[![Travis CI](https://img.shields.io/travis/inofix/ansible-acme-setup.svg?style=flat)](http://travis-ci.org/inofix/ansible-acme-setup)

Acme Setup
==========

This is an ansible role for setting up and preparing everything that is required for later signing certificates with let's encrypt - see inofix.acme-tiny-sign.

This role is meant to be run on any host that needs certificates.
If the host is not accessible via web - or does not use the inofix.acme-tiny-sign role for other reasons - a solution must be provided to transfer the cert-request forth and the final certificate back from this host to the acme-host. See inofix.acme-cron-proxy for an example.
Any host that signs certificates with the Let's Encrypt service, requires a web-server listening on port 80 and resolving /.well-known/acme-challenge to the directory accessible for the signing tool - see inofix.acme-sign for an example.

As this is actually the central role and the only role that is needed on any host involved, the following overview is provided here.
* A host that acts as a signing-host only and has itself no services that make use of the certificates will need to run these roles
  * inofix.acme-setup (this role)
  * inofix.acme-tiny-install
  * inofix.acme-tiny-sign
  * inofix.acme-tiny-cron-sign (to automatically repeat the signing once a month)
  * inofix.acme-proxy (to enable the remote host to get its certificates automatically)
* A host that makes only use of the certificates, but does not itself request the signing directly with Let's Encrypt will need to run these roles
  * inofix.acme-setup (this role)
  * inofix.acme-request
  * inofix.acme-cron-proxy (to automatically get the certificates from a remote host)
  * inofix.acme-cron-\<service\> (to restart the service if the certificate has changed)
* A host that runs both, a signing request tool and a certain service, will need these roles
  * inofix.acme-setup (this role)
  * inofix.acme-tiny-install
  * inofix.acme-request
  * inofix.acme-tiny-sign
  * inofix.acme-tiny-cron-sign (to automatically repeat the signing once a month)
  * inofix.acme-cron-\<service\> (to restart the service if the certificate has changed)

The development of this role was started as zwischenloesung.acme-tiny-setup.

Why we do not use one of the existing roles?

* For the first reason read the section "Promise" below. We need something reliable.
* This role will be used by [maestro](https://github.com/inofix/maestro) and must follow the logic used there. (Of course, the role can be used without maestro..)


State
-----

UNSTABLE! We are just migrating from zwischenloesung.acme-tiny-setup.


Promise
-------

Sure, this role may change in the future, but we will only expand features to not break backwards compatibility.

If radical changes should become necessary, a new role will be created, probably with an 'ng' or version suffix...

Installation
------------

 # ansible-galaxy install inofix.acme-setup

Requirements
------------

* Ansible >2.0
* Python2/3 on target host
* Generic UNIX with FHS
* Sudo
* Systemd (per default)

Role Variables
--------------

* app\_\_acme\_\_user - optional, default='acme'
* app\_\_acme\_\_group - optional, default='acme'
* app\_\_acme\_\_home - optional, default='/var/lib/acme'
* app\_\_acme\_\_config\_dir - optional, default='/etc/ssl/acme'
* app\_\_acme\_\_openssl\_config - optional, default='/etc/ssl/openssl.cnf'
* app\_\_acme\_\_challenge\_dir - optional, default='/var/www/acme-challenges'
* app\_\_acme\_\_scripts\_dir - optional, default='/etc/ssl/acme/scripts'
* app\_\_acme\_\_bin\_dir - optional, default='/usr/local/bin'
* app\_\_acme\_\_account\_key - optional, default='account.key'
* app\_\_acme\_\_domain - optional, default='example.com'
* app\_\_acme\_\_cert\_name - optional, auto
* app\_\_acme\_\_log\_dir - optional, default='/var/log/acme'
* app\_\_acme\_\_cert\_dir - optional, auto
* app\_\_acme\_\_key - optional, auto
* app\_\_acme\_\_request - optional, auto
* app\_\_acme\_\_letsencrypt\_certs - optional, default=[ {url='https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem', file='intermediate.crt'}, {url='https://letsencrypt.org/certs/isrgrootx1.pem', file='ca.crt'} ]
* app\_\_acme\_\_key\_length - optional, default=4096
* fqdn - optional, default={{ ansible\_fqdn | d(inventory\_hostname ) }}
* workdir - optional, default=/tmp (local dir, used for ssh-pub-key exchange)

Dependencies
------------


Example Playbook
----------------

    - hosts: servers
      roles:
         - inofix.acme-setup

License
-------

GPLv3


Author Information
------------------

* Michael Lustenberger at [inofix.ch](http://www.inofix.ch)
