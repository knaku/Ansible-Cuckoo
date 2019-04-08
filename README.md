## Synopsis
This playbook is intended for Ubuntu as the host OS for Cuckoo. Others distributions should work fine as well but some repositories changes are at the very least needed.
It has been has been successfully tested against a fresh install of 16.04 Desktop (amd64) and Ubuntu 17.10 Minimal (amd64) using Virtualbox. The playbook has also been tested with Ubuntu 18.04 Desktop (amd64) using VMware Workstation 15.

This is a fully functional updated standalone Ansible playbook for building a Cuckoo Sandbox server out of box. The following capabilities are included:

- Cuckoo Sandbox v2 inside virtual environment
- Yara v3
- Volatility v2
- Suricata IDS v4
- Java v8 and Elasticsearch v5
- Moloch v0.5
- Support for ESX, vSphere, KVM and AVD (with pre-configured avd image) machinery
- Current using VirtualBox v5 machinery with Extensions Pack
- Django-based Web Interface and MongoDB
- PostgreSQL
- SSDeep
- Tcpdump
- M2Crypto
- Tor


# Requirements
Ansible provisioner:
- A machine with ansible and its requirements installed, you can use [install_ansible.sh](install_ansible.sh) for installing ansible.
- You can also run the playbook locally on the Cuckoo Host but you will need to install ansible first and use staging instead of production. You can use [install_ansible.sh](install_ansible.sh) for installing ansible.

### Cuckoo Host:
A clean preconfigured Ubuntu machine with packages:
- openssh-server
- python

### Cuckoo Guest:
A preconfigured Windows or Linux machine fullfilling the [requirements](https://cuckoo.sh/docs/installation/guest/requirements.html) given by Cuckoo.
A quick summary for Windows:
- Installed python.
- agent.py that starts automatically, prefrably with administrative rights. agent.py can be started manually when creating the snapshot. It is not necessary for it to start automatically but it makes repeatablity a lot simpler. Use .pyw instead of .py if you want the terminal window to be minimised.
- Turn of the Windows firewall.
- Turn of Windows updating.
- Set a static IP. Cuckoo cannot handle DHCP yet. If you leave everything as default after cloning set it to 192.168.56.111 with a default gateway of 192.168.56.1.
- Snapshots:
  - You can create a snapshot manually while the VM is running with requirements above
  - You can let ansbile handle it automatically (mostly if using VMware Workstation). Agent.py is required to start on startup for this to work.
    - With Virtualbox you can let the playbook handle everything automatically.
    - With VMware Workstation you must manually start and stop the VM during the play if a snapshot does not already exists. If a snapshot exists it might create a bit of issue with Cuckoo and VMware.

# Preparations

### Cuckoo Host

 you cukoo host install the following packages:

- openssh-server
- python


You should also install all updates:

`apt update & apt dist-upgrade (or apt full-upgrade on Ubuntu 18=<)`


### Ansible provisioner

You need to have Ansible installed on your provisioner, either local or remote. You can use the [install_ansible.sh](install_ansible.sh) script if you want, Ubuntu only. To run locally use staging instead of production as inventory.

### Using Virtualbox
- Copy a  **cuckoo1.[ova/ovf]** file to the /home/ folder on your Cuckoo host machine. Using for example Rsync, ssh, sftp, hosting it on a server etc.
- Manually trasfer the ova/ovf to /home/ on the Cuckoo host.
The path set as default is /home/. This is set by the cuckoo_appliance variable in **cuckoo-playbook/inventories/production[or staging]/group_vars/all** file.

### Using VMware Workstation
As of now it will fail unless the VM folder is available for transfer with SSH. Location specified by variable vmx_location in **cuckoo-playbook/inventories/production[or staging]/group_vars/all** file.
- You can manually copy or download the vm folder to /home/cuckoo1/vmware/ after the cuckoo user is created. Remember to change it so the transfer task does not "failed_when".
- You can let the playbook handle it by using ssh. Painfully slow but it gets the work done.
- Host the VM on a server and download it manually to the Cuckoo host or adding a task to the playbook downloading it for you. Remember to change it so the transfer task does not "failed_when".
The path set as default is /home/cuckoo1/vmware/. This is set by the cuckoo_appliance_folder variable and cuckoo_user in **cuckoo-playbook/inventories/production[or staging]/group_vars/all** file.



# Installation

Installation of the Cuckoo environment is done with the following steps:

1. Clone this repository;
2. Replace the placeholders in **cuckoo-playbook/inventories/production/hosts** with the correct ones for your installation, where:
    - **HOST** is the IP address of the server to install Cuckoo to
    - **ADMIN** is a user with sudo privileges on the server
    - **PASSWORD** is the user _ADMIN_ password
4. Run the following command inside cuckoo-playbook folder:


##### Sidenote concerning VMware Workstation:
VMware is is installed using the trial link. If you have purchased VMware workstation you should consider switching out the download link with a that specifies a specific version of VMware Workstation to ensure continuity.


You can customize your target Ubuntu distro and the server network interface under (vmwareNetworkAdapter, user and license only needed for VMware):

    --extra-vars "distribution=bionic nic=ens32 vmwareNetworkAdapter=1 --license=xxxxx-xxxxx-xxxxx-xxxxx-xxxxx"

- "distribution" is the ubuntu distribution
- "nic" is the nic on the cuckoo host to use in the routing.conf
- "vmwareNetworkAdapter" is vmnet suffix used with the cuckoo guest. Only enter the value after vmnet. e.g for vmnet1: vmwareNetworkAdapter=1. vmnet1 is usually host-only while vmnet8 is usually NAT. If you don't set a custom vmnet it usually ends up using vmnet8. This is only necessary to add when using VMware workstation
- "license" is the license for VMware Workstation and is only  to add when using VMware workstation.

```
ansible-playbook -i inventories/production site.yml --extra-vars "distribution=bionic nic=ens32 vmwareNetworkAdapter=1 license=xxxxx-xxxxx-xxxxx-xxxxx-xxxxx"
```

By default, Cuckoo will be installed to `/opt/cuckoo` inside a virtual environment and a `cuckoo` user and group will be created. These values can be modified at **group_vars** file:

    cuckoo_user: 'cuckoo'
    cuckoo_dir: '/opt/cuckoo'

## Usage

Once the installation has completed, Suricata, Moloch, Cuckoo Rooter alongside API, Web interface and Cuckoo itself will start up automatically.

If you need to restart everything, enable `reboot_system` Ansible role by uncomment it, and comment the other roles, in the **site.yml** file and run the ansible-playbook again (we will enhance this in the future).

## Find yourself

- Cuckoo Instalation: http://docs.cuckoosandbox.org/en/latest/installation/
- CuckooDroid Instalation:
    - https://github.com/idanr1986/cuckoo-droid
    - https://github.com/idanr1986/cuckoodroid-2.0
- Cuckoo community modules: https://github.com/cuckoosandbox/community/tree/master/modules
- Ansible Instalation: https://docs.ansible.com/ansible/latest/intro_installation.html
- Guia em pt-BR de setup do Cuckoo + VM: [instalacao_do_cuckoo_sandbox.pdf](references/instalacao_do_cuckoo_sandbox.pdf)
- Tips to setup a guest victim VM can be found at [tips_to_prepare_win7_victim.md](references/tips_to_prepare_win7_victim.md)
- Check the VBoxHardenedLoader: https://github.com/hfiref0x/VBoxHardenedLoader


# Things that need manual changing if you change the default
- If the VM name is not cuckoo1 you will need to change entries in cuckoo.conf and { virutalizer }.conf
- If the VM snapshot is not cuckoo1-snapshot1 you will need to change entries in cuckoo.conf and { virutalizer }.conf
- If the {{ cuckoo_user }} is something else than "cuckoo" you will need to change entries in cuckoo.conf and { virutalizer }.conf
- If the path of the vm machine is something else than /home/cuckoo/vmware/cuckoo1/cuckoo1.vmx you will need to change entries in vmware.conf
- It might also be more that I have forgotten
