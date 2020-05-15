# ansible-playbooks
My collection of Ansible playbooks for various tasks.

Initially capturing the work I'm doing on Openshift installation, I will expand into other areas over time.

## Roles and playbooks
Here are the roles present so far.  Each role has a playbook that invokes it.

### configure-squid
Installs and configures Squid on a remote OCP Bastion system.  Configures permitted access from the OCP internal subnet.

### copy-playbooks
Takes a local copy of the 'multiarch-ci-playbooks' and sends them to the remote OCP Bastion system under construction.  

### create-guest-for-linux
Initiates the installation of a Linux guest under z/VM.  The following steps are performed:
* Uses z/VM SMAPI to define a guest for Linux
* creates a parameter file (parmfile)
* creates a Red Hat Kickstart configuration file
* punches files (kernel, parmfile, initramfs) to the new guest's reader
* Uses SMAPI to IPL the guest
* Uses SMAPI to set the IPL device to the guest's local disk

### create-local-ca
Uses Ansible OpenSSL support modules to generate a skinny Certificate Authority (CA) on the remote OCP Bastion host.

### set-groupvars
Patches the global_vars/all.yml file in the remote copy of the 'multiarch-ci-playbooks' for the values corresponding to the OCP environment being built.

### setup-ldap-tools
Installs and configures Apache, and downloads my RACF-ised versions of the LTB Project's "Service Desk" and "Self Service Password" tools.
