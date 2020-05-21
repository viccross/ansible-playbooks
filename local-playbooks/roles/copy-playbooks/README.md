copy-playbooks
==============

This role downloads the Openshift "multiarch-ci-playbooks" package to an OCP bastion host, and patches the group_vars/all.yml file for values that suit the candidate cluster.

Requirements
------------

Internet access, either direct or via a HTTP proxy, is required.  By default the role is configured for proxied HTTP.

Role Variables
--------------

The role inherits variables from the configurations of a Red Hat Openshift cluster.

Dependencies
------------

The Galaxy package "yedit" is required.

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
