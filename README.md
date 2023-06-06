# OpenShift on Equinix Baremetal

## Table of Contents

<!-- TOC -->

- [OpenShift on Equinix Baremetal](#openshift-equinix-baremetal)
  - [Introduction](#introduction)
  - [Prerequisitities](#prerequisities)
  - [Build the execution environment](#build-execution-environment)
  - [Configuration](#configuration)

<!-- TOC -->

## Introduction

Equinix provides baremetal resources on demand. This project enables you to automate the installation of OpenShift using Equinix baremetal instances as infrastructure
Since the BMC on Equinix is not reachable for using feature like virtualmedia, the only way to boot an ISO is using iPXE. OpenShift Assisted Installer will be used to install the cluster.
The ansible playbook will deploy the utility server and setup all the components such as Metal CLI, AICLI and iPXE webserver. At the end of the deployment the utility server is deleted to free up the resources.

## Prerequisities

   - Ansible
   - Ansible Builder
   - Equinix Metal API Token, Organization ID, Project ID
   - SSH public key/private key used to access Equinix instance

Clone this repository:

```
$ git clone https://github.com/tele0x/openshift-equinix-automation
$ cd openshift-equinix-automation
```

## Build the execution environment

We will use ansible-builder to build the execution environment in a container.
From the *openshift-equinix-automation* directory run:

```
$ ansible-builder build -t equinix-ocp-automation
```

## Configuration

There is several variables that has to be configured properly in order for the playbook to run correctly.

1. Edit the file *metal.yaml* and set the variables organization_id, project_id, token from Equinix Portal (how to get those is outside of scope of this document)
2. Add OpenShift pullsecret under *keys* directory and name the file *openshift_pull.json*
3. Add both SSH public key and private key under the *keys* directory, name the file ssh_public_key and ssh_private_key
    - IMPORTANT: change permission on private key: `chmod 600 keys/ssh_private_key`
4. Edit *group_vars/all*
    - Add again *api_token* and *project_id* from metal.yaml, needed by the equinix ansible collection
    - Set *ai_offlinetoken* to the OCM token from https://console.redhat.com/openshift/token 
    - Edit SNO clusters variables to change cluster name and base domain

Files in *keys* directory:

```
$ tree -l keys/
keys/
├── openshift_pull.json
├── ssh_private.key
└── ssh_public.key

0 directories, 3 files
```

## Installation

Execute the command and sit back and relax, once all it's completed your OpenShift Single Node cluster will be available on Equinix Baremetal.

```bash
$ podman run -v $(pwd):/runner/:Z --rm --name equinix localhost/equinix-ocp-automation ansible-playbook -vv --ssh-common-args='-o StrictHostKeyChecking=no' -i localhost playbooks/deploy.yaml

PLAYBOOK: deploy.yaml **********************************************************
3 plays in playbooks/deploy.yaml

PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
task path: /runner/playbooks/deploy.yaml:2
ok: [localhost]

TASK [(Equinix) Deploy utility server] *****************************************
task path: /runner/playbooks/deploy.yaml:10
changed: [localhost] => {"changed": true, "devices": [{"hostname": "utility", "id": "5303f1f3-7c47-4f23-bbf5-dddbeccf85da", "ip_addresses": [{"address": "147.28.148.195", "address_family": 4, "public": true}, {"address": "2604:1380:4641:f200::5", "address_family": 6, "public": true}, {"address": "10.70.65.5", "address_family": 4, "public": false}], "locked": false, "private_ipv4": "10.70.65.5", "public_ipv4": "147.28.148.195", "public_ipv6": "2604:1380:4641:f200::5", "state": "active", "tags": []}]}

TASK [[Equinix] Deploy equinixsno01 device] ************************************
task path: /runner/playbooks/deploy.yaml:20
changed: [localhost] => {"changed": true, "devices": [{"hostname": "equinixsno01", "id": "84dc7661-121b-4128-a81e-927188cc6bda", "ip_addresses": [{"address": "147.28.149.213", "address_family": 4, "public": true}, {"address": "2604:1380:4641:f200::1", "address_family": 6, "public": true}, {"address": "10.70.65.1", "address_family": 4, "public": false}], "locked": false, "private_ipv4": "10.70.65.1", "public_ipv4": "147.28.149.213", "public_ipv6": "2604:1380:4641:f200::1", "state": "active", "tags": []}]}
```
