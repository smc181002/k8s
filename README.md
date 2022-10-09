# Ansible Collection - smc181002.k8s

## Introduction

This is an ansible collection of roles that can be used to 
configure Kubernetes(k8s) cluster in any instance. The 
the collection consists of 4 roles named provision, common, 
master and slave 

> This is under development and can only serve for 
AWS' amazon Linux based os.

Components of the collection

```
roles
├─ provision
├─ common
├─ master
└─ slave
```

## `provision` role

The `provision` ansible role is made for the AWS to 
provision resources in AWS with the following 
parameters as `vars`

**`vars` Required**

* `key_name` - name of the key pair name to be used
* `sec_grp` - name of the security group in the default VPC 
* `ami_id` - AMI id of the instance (only works for Amazon 
Linux instances currently)
* `os_type` - name of the instance type (eg: t3.medium)

> To meet the k8s' resource requirements, for the cluster 
to function properly, provide at least 2vCPUs and 4Gi RAM

## `common` role

This is the role that runs after the provisioning of the 
instances. This role installs the docker container runtime
and iproute_tc for the networking required and installs 
`kubeadm`, `kubectl` and `kubelet` software for starting 
the cluster. This role also changes the cgroup driver to 
`systemd`.

> It is suggested to use the amazon's dynamic inventory as 
shown in the `examples/` as the provisioner adds the tags 
of Name: `kube_master` for master and `kube_slave` for 
slave and App: `kube` for both master and slave. These 
tags can be accessed using `tag_<TagKey>_<TagValue>`

## `Master` role

This role runs only in the master and first starts the 
cluster in the cloud and once the kubernetes master is 
working and provides information for the slaves to join.
For the joined slaves to have connectivity with each 
other, we use an overlay network called flannel that 
launches in the slaves as a `DaemonSet`. Flannel requires 
a network name to be provided and to change this from the 
default name using the `sed` command.

**`vars` Required**

- `net_name` - the network name for flannel to launch an 
overlay network

## `Slave` role

The master's join command output can be passed into the 
slave role's `vars` which is then executed in the slave 
nodes.

**`vars` Required**

- `join_command` - the command outputted by the master that
needs to run in the slave nodes
