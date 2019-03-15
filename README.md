## Synopsis

This is a fully functional updated standalone Ansible script for building a Cuckoo Sandbox server out of box. The following capabilities are included:

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

## Installation

This installation process has been success tested against a fresh install of _Ubuntu 16.04 Desktop (amd64), Ubuntu 17.10 Minimal (amd64) and Ubuntu 18.04 Desktop (amd64)_ with the following package options:

- openssh-server

Also, ensure a **cuckoo1.[ova/ovf]** file in your ADMIN - detailed below - home folder.
For VMware you as of now need to place the vmware folder with the VM in its own folder manually at root of the user cuckoo's home directory, will be handled automatically at a later point.

After the base OS install a dist-upgrade was conducted:

`apt update & apt dist-upgrade`

You may want to install Ansible with the [install_ansible.sh](install_ansible.sh) script, for Ubuntu only.

You can customize your target Ubuntu distro and the server network interface under (vmwareNetworkAdapter only needed for VMware):

    --extra-vars "distribution=artful nic=enp0s3 vmwareNetworkAdapter=vmnet1"

Installation of the Cuckoo environment is done with the following steps:

1. Clone this repository;
2. Replace the placeholders in **cuckoo-playbook/inventories/production/hosts** with the correct ones for your installation, where:
    - **HOST** is the IP address of the server to install Cuckoo to
    - **ADMIN** is a user with sudo privileges on the server
    - **PASSWORD** is the user _ADMIN_ password
4. Run the following command inside cuckoo-playbook folder:


vmwareNetworkAdapter is only needed if you use VMware as your virtualizer. It is only temporary for now.
```
ansible-playbook -i inventories/production site.yml --extra-vars "distribution=artful nic=enp0s3 vmwareNetworkAdapter=vmnet1"
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
