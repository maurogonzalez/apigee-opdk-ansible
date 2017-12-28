# Apigee OPDK ansible (Under construction)

This repo is intended to be a small and more friendly for non ansible users subset of [Carlos Frias ansible playbooks](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible) and [etp](https://github.com/yuriylesyuk/etp).

## Requirements
- ansible >= 2.2
- nmap
- node (to create topology diagram)
- etp (to create topology diagram)

#### Latest Releases Via Apt (Debian)
Add the following line to _/etc/apt/sources.list_:

`deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main`

Then run these commands:

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
$ sudo apt-get update
$ sudo apt-get install ansible
```

#### Latest Releases on Mac OSX

The preferred way to install ansible on a Mac is via pip.
```
$ sudo easy_install pip
$ sudo pip install ansible
```

- nmap

#### Latest Releases Via Apt (Debian)

```
$ sudo apt-get install nmap
```

#### Latest Releases on Mac OSX

```
$ brew install nmap
``` 

## Usage

#### Edge Topology Definition ([etp Edge Topology definition](https://github.com/yuriylesyuk/etp))
A topology of an Edge installation is represented as a JSON document, defining collections of nodes, subnets, and layout of the particular planet.

An example for a 5 node with 2 regions topology is given in `examples/topology.json`

#### Create an ansible inventory file 

- Create an [etp Edge Topology definition](https://github.com/yuriylesyuk/etp) (example `examples/topology.json`)
```
$ ansible-playbook -e "topology_src=topology.json" create_inventory.yml
```
- Find the response files under `inventory/inventory_{planet}.INI`

#### Create response files from inventory 

- Create an ansible inventory file based on [apigee-opdk-inventory-file](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible/blob/master/README-INVENTORY-FILE.md)
```
$ ansible-playbook -e -i inventory/{inventory_file} response_files.yml
```
- Find the response files under `files/response_{region}.cfg`

#### Create port checking report (Under construction) 

- Create port report (For now only CS)
```
$ ansible-playbook -i inventory/{inventory_file} port_report.yml
```
- Find the report files under `report/{component}_report.csv`