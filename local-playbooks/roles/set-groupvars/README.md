set-groupvars
=============

This role patches the group_vars/all.yml file in a downloaded Openshift "multiarch-ci-playbooks" package for values that suit the candidate cluster.

Requirements
------------

Not particularly :)

Role Variables
--------------

The role inherits variables from the configurations of a Red Hat Openshift cluster.

Dependencies
------------

The Galaxy package "yedit" is required.

Example Playbook
----------------

    - hosts: s390x_bastion_workstation
      roles:
         - { role: username.rolename, x: 42 }

License
-------

Apache-2.0

Author Information
------------------

Vic Cross - viccross@au1.ibm.com
