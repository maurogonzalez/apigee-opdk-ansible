# Apigee OPDK ansible (Under construction)

This repo is intended to be a small and more friendly for non ansible users subset of [Carlos Frias ansible playbooks](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible) and [etp](https://github.com/yuriylesyuk/etp). 

Centralize Apigee Edge operation tasks using one inventory file as reference for all tasks.

## Requirements
- ansible >= 2.2
- etp (to create topology diagram)

Go to the [installation wiki](https://github.com/maurogonzalez/apigee-opdk-ansible/wiki/Install-requirements).

# Usage

## Populate env.yml file
Under project root there's an _env.yml_, fill the required variables. 
```
license_path: LICENSE_PATH                    # license file absolute path including license file name.
apigee_user: APIGEE_SOFTWARE_USER             # software.apigee.com user.
apigee_pwd: APIGEE_SOFTWARE_PASSWORD          # software.apigee.com password.
ssh_user: SSH_USER                            # ssh user.
ssh_key: PATH_TO_KEY                          # ssh key absolute path.
ssh_pwd: SSH_PASSWORD                         # ssh password. If not needed, leave the original value.
ssh_bastion_host: BASTION_HOST                # ssh bastion/jumpbox host. If not needed, leave the original value.
ssh_bastion_user: BASTION_USER                # ssh bastion/jumbox user. If not needed, leave the original value.
ssh_bastion_key: PATH_BASTION_KEY             # ssh bastion/jumpbox key absolute path. If not needed, leave the original value.
pg_ram_in_mb: PG_NODE_RAM_IN_MB               # Postgresql machine memory required to set memory adjustments.
edge_version: 4.17.09                         # Apigee Edge version.
onboard_org_name: ORG_NAME                    # organization name for onboarding. If not needed, leave the original value. 
onboard_admin_username: ORG_ADMIN_USER        # org admin username (email format) for onboarding. If not needed, leave the original value. 
onboard_admin_name: ORG_ADMIN_NAME            # org admin name for onboarding. If not needed, leave the original value. 
onboard_admin_lastname: ORG_ADMIN_LASTNAME    # org admin lastname for onboarding. If not needed, leave the original value. 
onboard_admin_pwd: ORG_ADMIN_PASSWORD         # org admin password for onboarding. If not needed, leave the original value. 
onboard_env: ONBOARD_ENV                      # environment name for onboarding. If not needed, leave the original value. 
onboard_vhost_alias: ONBOARD_HOST_ALIAS       # virtualhost alias for onboarding. If not needed, leave the original value.
```

## Create an ansible inventory file and topology diagram

First create an [etp edge topology definition json file](https://github.com/yuriylesyuk/etp) (example _examples/topology-1dc-5n.json_).
This is the only file that you need to create.

Ansible inventory file based on [apigee-opdk-inventory-file](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible/blob/master/README-INVENTORY-FILE.md).

Create an inventory file used by all Ansible playbooks and operation tasks:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" inventory.yml
```
Optionally, you can create the topology diagram in the same run setting the diagram variable to any value*:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE diagram=1" inventory.yml
```
Create only the diagram*:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" diagram.yml
```
Find the files under:
  - _inventory/inventory\_PLANET.INI_
  - _reports/topology-PLANET.svg_

**Notes:** 
  - \* These playbooks, diagram tasks, require nodejs and etp.
  - Do not modify the inventory file.

## Create response files from inventory 

Create response file per region used by the installation, upgrade and other ops tasks. Also
create the onboarding response file:
```
$ ansible-playbook -e -i inventory/INVENTORY_FILE response_files.yml
```
Find the response files under:
  - _files/response_PLANET_REGION.cfg_

## Install all prereqs and the apigee-setup utility accross the planet
Install the prerequisites described [here](https://docs.apigee.com/private-cloud/latest/install-edge-apigee-setup-utility).
Install the **apigee-setup** utility across the planet and the **apigee-provision** utility in the MS nodes.
Create the Cassandra and Message Processor systemlimits files, and the Postgresql memory settings file. 
Upload the license file and the response files across the planet.

```
$ ansible-playbook -i inventory/INVENTORY_FILE prerequisites.yml
```

## Create port checking report

Create port report for a whole planet. This will test ports between nodes and create two CSV files:
- Connectivity report grouped by edge component.
- Compact report grouped by host.
```
$ ansible-playbook -i inventory/INVENTORY_FILE port_report.yml
```
Find the report files under: 
  - `reports/port_connectivity_report_PLANET.csv`
  - `reports/port_compact_PLANET.csv`

**Note:** The hosts require _nmap_, it is installed in the above prerequisites playbook.
## Install Apigee Edge in planet
Install apigee edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE -e "cmd=setup" setup.yml
```
## Edge onboarding
Onboarding: create organization, environment, org admin user and default virtual host.

```
$ ansible-playbook -i inventory/INVENTORY_FILE onboard.yml
```

## Create Keystore and upload keystore jar
Create a JAR Keystore from key/cert pair, a Keystore in Edge and upload the JAR to Edge:

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "keyalias=KEY_ALIAS keystore=KEYSTORE_NAME \
  ks_cert=PATH_TO_CERT ks_key=PATH_TO_KEY \
  ks_org=EDGE_ORG ks_env=EDGE_ENV" \
  keystore.yml
```

## Create a VirtualHost with TLS enabled
Create virtual host and if **tls_enabled** set to any value in extra vars is created with TLS/SSL:

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "keyalias=KEY_ALIAS keystore=KEYSTORE_NAME \
  vhost_name=VHOST_NAME vhost_aliases=COMMA_SEPARATED_ALIASES \
  org=EDGE_ORG env=EDGE_ENV tls_enabled=true" \
  vhost_tls.yml
```

## Run apigee-service commands accross planet
Run an apigee-service command in particular components or across the planet.

Possible values for command:
- status
- start
- wait_for_ready
- stop
- restart

Possible values for component:
- zk      (Zookeeper)
- cs      (Cassandra)
- ds      (Zookeeper and Cassandra)
- ldap    (OpenLDAP)
- ms      (Management Server)
- msldap  (Management Server and OpenLDAP)
- r       (Router)
- mp      (Message Processor)
- rmp     (Router and Message Processor)
- qs      (QPIDD and Qpid Server)
- pg      (Postgreql and Postgres Server)
- all     (All Apigee Edge components)

```
$ ansible-playbook -i inventory/INVENTORY_FILE -e "cmd=COMMAND component=COMPONENT" setup.yml
```

## Fetch Planet logs
Tar all the edge logs from each node.

```
$ ansible-playbook -i inventory/INVENTORY_FILE logs.yml
```

Find the logs under:
  - _reports/PLANET/_:

## Ansible Ad-Hoc commands
[Ansible Ad-Hoc commands](http://docs.ansible.com/ansible/latest/intro_adhoc.html)

Example: 
```
$ ansible -i inventory/INVENTORY_FILE HOST_GROUP -m shell -a '/opt/apigee/apigee-service/bin/apigee-all status'
```
Script for inventory group hosts:
Run _echo "Hello node!"_ command example:
```
$ ./service.sh -i inventory/INVENTORY_FILE -g HOST_GROUP -c "echo 'Hello node!'"
```
Where:
- **-h**: help
- **-i**: ansible inventory path **Required**
- **-c**: command to run in the node.
- **-g**: ansible inventory group

Default values:
- If not _-c_ option provided, the default command is: _/opt/apigee/apigee-service/bin/apigee-all status_
- If not _-g_ option provided, the default group is: _planet_
```
$ ./service.sh -i inventory/INVENTORY_FILE -g HOST_GROUP -c "echo 'Hello node!'"
```
## Author

If you have any questions regarding this project contact:  
Mauro González <jmajma8@gmail.com>

