configure-squid
===============

This role installs and configures Squid.

Requirements
------------

The configured host needs YUM configured for the installation to complete.

Role Variables
--------------

The role inherits variables from the configurations of a Red Hat Openshift cluster.

Dependencies
------------

No dependencies.

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
