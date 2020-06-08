# ansible-playbooks
My collection of Ansible playbooks for various tasks.

Initially capturing the work I'm doing on Openshift installation, I will expand into other areas over time.

# Openshift
Currently, the Openshift "[multiarch-ci-playbooks](https://github.com/multi-arch/multiarch-ci-playbooks)" package provides the commonly-used set of playbooks for building an OCP cluster for testing or demonstration on z/VM on IBM Z/LinuxONE. However there is a reasonable amount of manual work outside these playbooks; for example defining z/VM guests for the CoreOS nodes in the cluster, and kicking-off the installation process.

Some of the playbooks here represent an approach to "wrap" the existing Ansible playbooks and supplement them to provide additional functions.  It is also intended that these wrapping functions prepare a system for the role of OCP Bastion host for later deployment, possibly on a different site. 

A single Ansible playbook, `build-a-bastion.yml`, has been created to build a new OCP Bastion host.  To use it, create a new Ansible host definition (or new inventory) for the cluster and bastion host to be built.  Then, the playbook will run the following in sequence:

* local playbook `create-guest-for-linux` to build a guest for use as the OCP Bastion host
* if required (e.g. if the OCP cluster is on an isolated network), local playbook `configure-squid` to set up Squid on the OCP Bastion host
* local playbook `create-local-ca` to generate a local Certificate Authority for the environment
* local playbook `setup-ldap-tools` to install and configure the Apache server and install and configure the PHP LDAP Tools (more info about the tools is here: [Self-Service Password](https://github.com/viccross/self-service-password-racf) and [LDAP Service Desk](https://github.com/viccross/service-desk-racf)
* "m-c-p" playbook `configure-dns` on the OCP Bastion host, to set up the DNS for the cluster
* local playbook `configure-haproxy` on the OCP Bastion host, to set up the TCP balancer HA-Proxy
* local playbook `enable-haproxy-stats` on the OCP Bastion host, to turn on HA-Proxy's statistics and monitor interface
* local playbook `configure-apache-ignition` on the OCP Bastion host, to download CoreOS files and prepare for cluster build
* local playbook `setup-ocp-deployer` to install the playbooks (including appling the desired OCP cluster configuration to the variables in the copied playbook inventory) on the OCP Bastion and finalise all the setup.

At this point the new OCP Bastion host is ready to be used to build the OCP cluster. If the OCP cluster is being built at a different location, the OCP Bastion host can be transported to the other site. When the OCP Bastion host is where it needs to be, the installation of the cluster can be finalised:

* local playbook `create-cluster` to set up the z/VM guests for the cluster, start the CoreOS installation in them, and to finalise all the dependencies and run `openshift-install` to build the cluster

## Roles and playbooks
Here are the Openshift-related roles present so far.  Each role has a playbook that invokes it.

### configure-apache-ignition
A localised version of the playbook with the same name from "m-c-p".  Key difference being that instead of creating parmfiles, it generates files to be used with the [ZNETBOOT](https://github.com/trothr/znetboot) tool.  ZNETBOOT greatly simplifies the initial bootstrap of Linux from installation media.  We are using ZNETBOOT to simplify the creation of the OCP cluster guests (doing it the 'traditional' way is possible, but complex and difficult to automate with Ansible).

### configure-haproxy
A localised version of the playbook with the same name from "m-c-p".  This version is a tidy-up from the original, and includes a community patch to the systemd service file to avoid a boot-time problem with starting HA-Proxy.

### configure-internal-net
Configures the internal (private) network interface on the OCP Bastion host.  Changed to configuring the interface as part of the Kickstart installation of the guest, so this is not used in `build-a-bastion` right now.

### configure-squid
Installs and configures Squid on a remote OCP Bastion system.  Configures permitted access from the OCP internal subnet.

### copy-playbooks
Takes a local copy of the 'multiarch-ci-playbooks' and sends them to the remote OCP Bastion system under construction.  Not used in `build-a-bastion` right now, since `setup-ocp-deployer` generates a more tailored playbook than the full set from "m-c-p".

### create-cluster
Originally from "m-c-p", this is the template for the reduced playbook that gets deployed to the Bastion host.  The static tasks have been moved to `setup-ocp-deployer` for execution prior to the OCP Bastion being deployed to the final site.

### create-guest-for-linux
Initiates the installation of a Linux guest under z/VM.  Version 1 did the following steps:
* Uses z/VM SMAPI to define a guest for Linux
* creates a parameter file (parmfile)
* creates a Red Hat Kickstart configuration file
* punches files (kernel, parmfile, initramfs) to the new guest's reader
* Uses SMAPI to IPL the guest
* Uses SMAPI to set the IPL device to the guest's local disk

In developing to support installation on a remote z/VM system (i.e. a z/VM system that is not the same as the Ansible controller runs under), it was changed to use the [ZNETBOOT](https://github.com/trothr/znetboot) tool.  So the current steps are:
* Uses z/VM SMAPI to define a guest for Linux
* creates a ZNETBOOT configuration file (`guestname ZNETBOOT`)
* sends the ZNETBOOT file to a prepared shared disk via FTP (`curl -T`)
* Uses SMAPI to IPL the guest
* Uses SMAPI to set the IPL device to the guest's local disk

### create-local-ca
Uses Ansible OpenSSL support modules to generate a skinny Certificate Authority (CA) on the remote OCP Bastion host.

### enable-haproxy-stats
Enables the HA-Proxy HTML status page.  Can help troubleshooting a cluster build, or otherwise just provide info on the services and APIs within the cluster.

### set-groupvars
Patches the global_vars/all.yml file in the remote copy of the 'multiarch-ci-playbooks' for the values corresponding to the OCP environment being built.  Not currently called by `build-a-bastion` right now, since `setup-ocp-deployer` generates a more tailored inventory file with fewer variables.

### setup-ldap-tools
Installs and configures Apache, and downloads my RACF-ised versions of the LTB Project's "Service Desk" and "Self Service Password" tools.

### setup-ocp-deployer
Where all the final setup of the OCP Bastion is done.  This task:
* Installs the EPEL repository (including patching YUM/DNF configuration for HTTP proxy, if defined)
* Installs Ansible
* Sets up directories for the playbooks and inventory to be copied
* Downloads `openshift-install` and `oc`
* Generates an SSH key for access into the CoreOS nodes
* Generates the template for the Openshift `install-config.yaml` file
* Generates the template for the OCP cluster guests to be deployed
* Copies the Ansible playbook, and dependent templates etc
* Copies the `smcli` utility for communicating to SMAPI on the destination z/VM
* Copies DASD configuration script, and installs as a boot-time service
* Creates the Ansible inventory for the cluster

## Dependencies
The playbooks/roles have a few dependencies.

### `smcli`
I am using a 'historical' version written originally by Leland Lucius, found [here](https://homerow.net/zvm/smcli/).  Many of the APIs introduced since z/VM 6.3 are not supported in this tool, but I have not found a need for them yet.  There are other SMAPI command-line tools these days, including one published by an IBM project as part of the [IBM Cloud Connector](https://cloudlib4zvm.readthedocs.io).  I like `smcli` because it is a single Bash script that does what I need, however I acknowledge that at some point I may need to migrate to the Cloud Connector one.

### ZNETBOOT
While it makes the Asnpible task definition much simpler, ZNETBOOT creates a dependency.  Not only is the ZNETBOOT code required, but it has to be installed on the destination z/VM system.  On the roadmap is a playbook/task to deploy the ZNETBOOT dependency into the z/VM system as part of the Ansible tooling (see later).

### Destination z/VM dependencies
The z/VM being deployed into needs to have the following:
* System Management APIs (SMAPI) configured for IPv4 access (VSMREQIN)
* IBM Directory Maintenance Facility (DirMaint) is assumed.  While all directory operations are handled by SMAPI, the guests are created using a 'template' which is 'z/VM directory' syntax.  I am not aware of whether other directory managers support the same syntax for defining a guest, and I can't test.
* A set of large (EAV) 3390 DASD for the OCP guests to be installed to.  They do not have to be attached to the system or defined to DirMaint, but they *do* need to be labelled with a unique prefix (a service is installed onto the Bastion that locates the DASDs, attaches them, and defines them to DirMaint if needed).

## Issues / To-Do
It's not perfect... yet  :)

### ZNETBOOT dependency
There is a hard-coded dependency on a "MAINT 200" disk that holds the ZNETBOOT files.  The ZNETBOOT code, a `PROFILE EXEC` executed by the booting guests, and the ZNETBOOT configuration files, all reside (or will be created on) this disk.  At present the creation of this disk and the initial setup of the ZNETBOOT code is assumed to have been done externally.  This needs to be brought into the tooling.

### z/VM TLS/SSL
The reasoning for creating a local CA on the bastion is to generate certificates for all services inside the environment that need securing, so the client will need to download and install only the CA certificate for them all to be secured.  While it might be easier to use a string of self-signed certificates (or worse, to not use SSL/TLS at all) for simplicity, it's important that we demonstrate as secure an environment as possible.  At present, only the Apache instance that serves the LDAP Tools has a server cert created using the local CA and installed.  This needs to be extended to include:
* OCP Bastion Cockpit instance
* z/VM TCP/IP TN3270 and FTP
* z/VM LDAP
* OCP cluster dashboard (if feasible)

### Idempotence
One of the anchors of Ansible is the concept of __idempotence__.  An action should only be taken if it is required, and an Ansible task executed repeatedly will not result in unpredicted (or unneccesary) changes in state.  Some of the operations of these playbooks is not idempotent however, because they are invoked using shell commands.  Development of a z/VM "module" for Ansible would help this greatly.

