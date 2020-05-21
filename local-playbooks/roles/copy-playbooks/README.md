copy-playbooks
==============

This role copies a downloaded Openshift "multiarch-ci-playbooks" package to an OCP bastion host.

Requirements
------------

The Openshift "multiarch-ci-playbooks" package must be already downloaded, at the specified location, on the Ansible controller.

Role Variables
--------------

None.

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
