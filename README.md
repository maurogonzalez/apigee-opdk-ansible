# Apigee OPDK ansible (Under construction)

This repo is intended to be a small and more friendly for non ansible users subset of  [Carlos Frias ansible playbooks](https://github.com/carlosfrias/apigee-opdk-playbook-setup-ansible).

## Requirements
- ansible >= 2.2
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
$ brew install pip
``` 

## Usage

```
$ ansible-playbook {playbook_file} -i inventory/{inventory_file}
```