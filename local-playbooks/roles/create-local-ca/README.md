create-local-ca
===============

This role uses the built-in Ansible OpenSSL support to create a local 'skinny' Certificate Authority.  The intention is to simplify deployment of SSL/TLS-enabled applications by kicking-off a simple way to produce certificates.

Requirements
------------

Not particularly :)

Role Variables
--------------

The role inherits variables from the configurations of a certificate authority [details soon].

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: s390x-bastion-workstation
      roles:
         - { role: username.rolename, x: 42 }

License
-------

Apache-2.0

Author Information
------------------

Vic Cross - viccross@au1.ibm.com
