# ansible-playbooks
My collection of Ansible playbooks for various tasks.

Initially capturing the work I'm doing on Openshift installation, I will expand into other areas over time.

## Openshift
Currently, the Openshift "[multiarch-ci-playbooks](https://github.com/multi-arch/multiarch-ci-playbooks)" package provides the commonly-used set of playbooks for building an OCP cluster for testing or demonstration on z/VM on IBM Z/LinuxONE. However there is a reasonable amount of manual work outside these playbooks; for example defining z/VM guests for the CoreOS nodes in the cluster, and kicking-off the installation process.

Some of the playbooks here represent an approach to "wrap" the existing Ansible playbooks and supplement them to provide additional functions.  It is also intended that these wrapping functions prepare a system for the role of OCP Bastion host for later deployment, possibly on a different site. 

At current point in development, the sequence to build a new OCP Bastion host would be:

* create a new Ansible host definition (or new inventory) for the cluster to be built
* local playbook `create-guest-for-linux` to build a guest for use as the OCP Bastion host
* local playbook `copy-playbooks` to install the upstream playbooks
* local playbook `set-groupvars` to apply the desired OCP cluster configuration to the variables in the copied playbook inventory
* if required (e.g. if the OCP cluster is on an isolated network), local playbook `configure-squid` to set up Squid on the OCP Bastion host
* "m-c-p" playbook `configure-dns` on the OCP Bastion host, to set up the DNS for the cluster
* "m-c-p" playbook `configure-haproxy` on the OCP Bastion host, to set up the TCP balancer
* "m-c-p" playbook `configure-apache-ignition` on the OCP Bastion host, to download CoreOS files and prepare for cluster build

At this point the new OCP Bastion host is ready to be used to build the OCP cluster. If the OCP cluster is being built at a different location, the OCP Bastion host can be transported to the other site. When the OCP Bastion host is where it needs to be, the installation of the cluster can be finalised:

* local playbook `create-ocp-cluster nodes` on the OCP Bastion host to set up the z/VM guests for the cluster, and to start the CoreOS installation in them
* "m-c-p" playbook `create-cluster` to finalise all the dependencies and run `openshift-install` to build the cluster

Work is underway to pull these separate stages into as few steps as possible, either by having one or two scripts to call the Ansible work or by building new playbooks from the existing roles.

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
