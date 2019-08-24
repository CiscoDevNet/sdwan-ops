# sdwan-ops

This repo is a derivative of the full CI/CD Pipeline described in the [sdwan-devops](https:github.com/ciscodevnet/sdwan-devops) aimed just at daily operational tasks.  It is based on the [Cisco SD-WAN Ansible Modules](https:github.com/ciscodevnet/ansible-viptela).

## Cloning this repository

The repository contains the Cisco SD-WAN Ansible Modules as a submodule.  To clone the repo with the submodule:
``` bash
git clone --recursive https:github.com/ciscodevnet/sdwan-ops
```
## Requirements
* ansible
* requests

### Installing requirements with pip
```
pip install -r requirements.txt
```

## Playbooks

These playbooks all take the following extra-vars:
* `vmanage_host`: The IP address or DNS name of vmanage
* `vmanage_user`: User with adminsitrator privileges on vmanage
* `vmanage_password`: Password for the user specified

>Note: These vars can also be set in inventory

### Device Facts

Display facts about the devices in vmanage (e.g. controllers, out of sync edges, free devices, etc.).  This can be
used as the basis for other playbooks

``` bash
ansible-playbook device-facts.yml -e vmanage_host=1.2.3.4 -e vmanage_user=admin -e vmanage_password=admin
```

### Export/Import Policy

The following playbooks will export both templates and policy from vmanage in there entirety into a JSON file.  In addition, they will import
both templates and policy from a JSON file into vmanage, adding only the templates and/or policy that is not already present (i.e. idempotent)

> Extra Vars:
> * `file`: the name of the file to either export to or import from.

#### Export Templates from vmanage

``` bash
ansible-playbook export-templates.yml -e vmanage_host=1.2.3.4 -e vmanage_user=admin -e vmanage_password=admin
```

#### Export Policy from vmanage

``` bash
ansible-playbook export-policy.yml -e vmanage_host=1.2.3.4 -e vmanage_user=admin -e vmanage_password=admin
```

#### Import Templates to vmanage

``` bash
ansible-playbook import-templates.yml -e vmanage_host=1.2.3.4 -e vmanage_user=admin -e vmanage_password=admin
```

#### Import Policy to vmanage

``` bash
ansible-playbook import-policy.yml -e vmanage_host=1.2.3.4 -e vmanage_user=admin -e vmanage_password=admin
```