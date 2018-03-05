[![Travis CI](https://img.shields.io/travis/inofix/ansible-acme-setup.svg?style=flat)](http://travis-ci.org/inofix/ansible-acme-setup)

Acme Setup
==========

This is an ansible role for setting up and preparing everything that is
required for later signing certificates with let's encrypt, i.e.
installing the user, and relevant directories and scripts, and setting
the permissions right.

This role is meant to be run on any host that needs certificates for itself
or signs any for some other host.

See inofix.acme-request for the creation of the private key and the
certificate requests resp. the installation of a remotely created
request on a "signing" or "certificate-proxy" host.

The inofix.acme-tiny-install role installes the acme-tiny script on
"signing" hosts.

The inofix.acme-tiny-sign / inofix.acme-tiny-cron resp. roles do care
for the actual (resp. periodical) signing on the "signing" hosts.

Any host that signs certificates with the Let's Encrypt service, requires a web-server listening on port 80 and resolving /.well-known/acme-challenge to the directory accessible for the signing tool - see inofix.acme-sign for an example.

Hosts relying on other hosts to do the signing for them (e.g. they
have no webservice installed) can access and transfere the ready
signed certs from their proxy host with inofix.acme-proxy.

See "Overview / Concept" below for details.

The development of this role was started as zwischenloesung.acme-tiny-setup.

Why we do not use one of the existing roles?

* For the first reason read the section "Promise" below. We need something reliable.
* This role will be used by [maestro](https://github.com/inofix/maestro) and must follow the logic used there. (Of course, the role can be used without maestro..)


State
-----

preSTABLE (Feature-Freeze/RC)


Promise
-------

Sure, this role may change in the future, but we will only expand features to not break backwards compatibility.

If radical changes should become necessary, a new role will be created, probably with a version suffix...

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


Overview / Concept
------------------

As this is actually the central role and the only role that is needed on any host involved, an overview is provided in this README. 


_Role Perspective_

* inofix.acme-setup
  * run on any host
  * set up the environement
    * create user 'acme'
    * create keys etc
      * /etc/ssl/acme
        * scripts
        * connected services
      * /var/log/acme
        * logs
      * /var/lib/acme
        * home for user signing and copying certs
* inofix.acme-request
  * run on hosts using or signing a cert
  * on hosts using a cert
   * create the private key
   * create a certificate-request (csr)
  * on proxy hosts
   * install the csr from "offline" hosts
* inofix.acme-tiny-install
  * run on hosts signing certs with Let's Encrypt
  * install the acme-tiny.py script
* inofix.acme-tiny-sign
  * run on hosts signing certs with Let's Encrypt
   * only needs the csr: the private key is not even read (or absent for proxy)
  * use acme-tiny.py to ask Let's Encrypt to sign a cert for the csr
* inofix.acme-tiny-cron
  * run on hosts signing certs with Let's Encrypt
   * only needs the csr: the private key is not even read (or absent for proxy)
  * install a cron job to do the signing part (as in inofix.acme-tiny-sign)
* inofix.acme-proxy
  * run on hosts using a cert but not doing the signing themselves (e.g. mail/jabber/..)
  * copy the cert from a remote host (that did the signing)
* inofix.acme-service-..
  * run on all hosts using a cert (i.e. running a certified service...)
  * register a service to be restarted if any cert has changed


_Host Perspective_

* A host that acts as a signing-host only and has itself no services that make use of the certificates will need to run these roles
  * inofix.acme-setup (this role)
  * inofix.acme-tiny-install
  * inofix.acme-tiny-sign
  * inofix.acme-tiny-cron (to automatically repeat the signing once a month)
* A host that makes only use of the certificates, but does not itself request the signing directly with Let's Encrypt will need to run these roles
  * inofix.acme-setup (this role)
  * inofix.acme-request
  * inofix.acme-proxy (to automatically get the certificates from a remote host)
  * inofix.acme-service-\<service\> (to restart the service if the certificate has changed)
* A host that runs both, a signing request tool and a certain service, will need these roles
  * inofix.acme-setup (this role)
  * inofix.acme-tiny-install
  * inofix.acme-request
  * inofix.acme-tiny-sign
  * inofix.acme-tiny-cron (to automatically repeat the signing once a month)
  * inofix.acme-service-\<service\> (to restart the service if the certificate has changed)


Role Variables
--------------

* app\_\_acme\_\_user - optional, default='acme'
* app\_\_acme\_\_group - optional, default='acme'
* app\_\_acme\_\_home - optional, default='/var/lib/acme'
* app\_\_acme\_\_config\_dir - optional, default='/etc/ssl/acme'
* app\_\_acme\_\_openssl\_config - optional, default='/etc/ssl/openssl.cnf'
* app\_\_acme\_\_challenge\_dir - optional, default='/var/www/acme-challenges'
* app\_\_acme\_\_scripts\_dir - optional, default='/etc/ssl/acme/scripts'
* app\_\_acme\_\_service\_dir - optional, default='/etc/ssl/acme/service.d'
* app\_\_acme\_\_bin\_dir - optional, default='/usr/local/bin'
* app\_\_acme\_\_account\_key - optional, default='account.key'
* app\_\_acme\_\_key\_length - optional, default=4096
* app\_\_acme\_\_ssh\_keytype - optional, default='rsa'
* app\_\_acme\_\_log\_dir - optional, default='/var/log/acme'
* app\_\_acme\_\_letsencrypt\_certs - optional, default=[ {url='https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem', file='intermediate.crt'}, {url='https://letsencrypt.org/certs/isrgrootx1.pem', file='ca.crt'} ]
* app\_\_acme\_\_cron\_minute - optional, default='11'
* app\_\_acme\_\_cron\_hour - optional, default='5'
* app\_\_acme\_\_cron\_day - optional, default='\*'
* app\_\_acme\_\_cron\_month - optional, default='\*'
* app\_\_acme\_\_cron\_year - optional, default='\*'
* fqdn - optional, default={{ ansible\_fqdn | d(inventory\_hostname ) }}
* workdir - optional, default=/tmp (local dir, used for ssh-pub-key exchange)

Dependencies
------------


Example Playbook
----------------

    - hosts: servers
      roles:
         - inofix.acme-setup

See https://raw.githubusercontent.com/inofix/common-playbooks/master/setup-lets-encrypt.yml for a complete playbook with all the relevant roles included.

License
-------

GPLv3


Author Information
------------------

* Michael Lustenberger at [inofix.ch](http://www.inofix.ch)
