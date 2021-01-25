configure-dnsmasq
=================

Install and configure dnsmasq as the DNS for an OCP Bastion host.

Requirements
------------

**Further testing!** There are no system requirements, but the playbook has not been extensively tested.

Role Variables
--------------

IP address variables `bastion_public_ip_address`, `bastion_private_ip_address`, and `cluster_domain_name` are requirements.

The z/VM IP address will be added.

Dependencies
------------

No dependencies outside base Ansible.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```
    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }
```
License
-------

Apache-2.0

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
