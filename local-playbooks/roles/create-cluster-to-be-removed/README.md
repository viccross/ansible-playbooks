Role Name
=========

Modified version of the create-cluster role from multiarch-ci-playbooks.  This one is designed to be deployed into the OCP Bastion (by the `setup-ocp-deployer` role) with most/all of the variables for the new cluster being populated in advance.

*** To avoid duplication this is being moved to become templates under the `setup-ocp-deployer` role (where most of the content already is anyway).

Requirements
------------

The setup work for the OCP Bastion needs to be performed before this.  The playbook build-a-bastion does this, or the individual roles can be run.

Role Variables
--------------

Too many to mention!  In other words, coming soon :)

Dependencies
------------

None at this time.

Example Playbook
----------------

    - hosts: s390x_bastion_workstation
      roles:
         - { roles/create-cluster }

License
-------

Apache-2.0 (need to confirm the license on multiarch-ci-playbooks though).

Author Information
------------------

Vic Cross
https://github.com/viccross
mailto:viccross@au1.ibm.com
