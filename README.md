# Apigee OPDK ansible (Under construction)

This repo is intended to be a small and more friendly for non ansible users subset of [Carlos Frias ansible playbooks](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible) and [etp](https://github.com/yuriylesyuk/etp).

## Requirements
- ansible >= 2.2
- nmap
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

# Usage

## Populate env.yml file
Under project root there's an _env.yml_, fill the required variables:
```
license_path: LICENSE_PATH
apigee_user: APIGEE_SOFTWARE_USER
apigee_pwd: APIGEE_SOFTWARE_PASSWORD
ssh_user: SSH_USER
ssh_key: PATH_TO_KEY
ssh_pwd: SSH_PASSWORD
ssh_bastion_host: BASTION_HOST
ssh_bastion_user: BASTION_USER
ssh_bastion_key: PATH_BASTION_KEY
pg_ram_in_mb: PG_NODE_RAM_IN_MB
edge_version: 4.17.09
onboard_org_name: ORG_NAME
onboard_admin_username: ORG_ADMIN_USER
onboard_admin_name: ORG_ADMIN_NAME
onboard_admin_lastname: ORG_ADMIN_LASTNAME
onboard_admin_pwd: ORG_ADMIN_PASSWORD
onboard_env: ONBOARD_ENV
onboard_vhost_alias: ONBOARD_HOST_ALIAS
```

## Create an ansible inventory file and topology diagram

First create an [etp edge topology definition json file](https://github.com/yuriylesyuk/etp) (example _examples/topology-1dc-5n.json_).

Create inventory file:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" inventory.yml
```
Optionally, you can create the topology diagram in the same run setting the diagram variable to any value:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE diagram=1" inventory.yml
```
Create only the diagram:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" diagram.yml
```
Find the files under:
  - _inventory/inventory\_PLANET.INI_
  - _reports/topology-PLANET.svg_

## Create response files from inventory 

Create an ansible inventory file based on [apigee-opdk-inventory-file](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible/blob/master/README-INVENTORY-FILE.md):
```
$ ansible-playbook -e -i inventory/INVENTORY_FILE response_files.yml
```
Find the response files under `files/response_PLANET_REGION.cfg`

## Create port checking report

Create port report for a whole planet using an inventory
```
$ ansible-playbook -i inventory/INVENTORY_FILE port_report.yml
```
Find the report files under `reports/port_connectivity_report_PLANET.csv`

## Install all prereqs and the apigee-setup utility accross the planet
Install the apigee-setup utility and create the CS and MP systemlimits files and PG memory settings file:

```
$ ansible-playbook -i inventory/INVENTORY_FILE prerequisites.yml
```

## Install Apigee Edge in planet
Install apigee edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE setup.yml
```
## Edge onboarding
Install apigee edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE onboard.yml
```

## Create Keystore and upload keystore jar
Install apigee edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "keyalias=KEY_ALIAS keystore=KEYSTORE_NAME \
  ks_cert=PATH_TO_CERT ks_key=PATH_TO_KEY \
  ks_org=EDGE_ORG ks_env=EDGE_ENV" \
  keystore.yml
```

## Fetch Planet logs
Install apigee edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE logs.yml
```

## Author

If you have any questions regarding this project contact:  
Mauro González <jmajma8@gmail.com>

