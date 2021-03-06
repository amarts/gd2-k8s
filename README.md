# GCS Deployment scripts

This repository contains playbooks to deploy a GCS cluster on Kubernetes. It also contains a Vagrantfile to setup a local GCS cluster using vagrant-libvirt.

> IMP: Clone this repository with submodules

## Available playbooks

### deploy-k8s.yml

This playbook deploys a kubernetes cluster on the configured nodes, and creates a copy of the kube-config to allow kubectl from the Ansible host.
Kubernetes is configured to use Flannel as the network plugin.

TODO: Describe the minimum required ansible inventory format. Till then ./kubespray/inventory/sample/hosts.ini


### deploy-gcs.yml

This playbook deploys GCS on a Kubernetes cluster. All of GCS, except for the StorageClass is deployed into its own namespace. The playbook deploys the following,

- etcd-operator
- etcd-cluster with 3 nodes
- glusterd2-cluster with a pod on each kube node and configured to use the deployed etcd-cluster
- glusterd2-client service providing a single rest access point to GD2
- csi-driver (provisioner, nodeplugin, attacher) configured to use glusterd2-client service to reach GD2

Uses the inventory defined for deploy-k8s.yml.

At the moment this is a playbook, but it really shuold become a role.
TODO: Convert to role
TODO: Add more configurability

### add-devices.yml

This playbook adds block devices to the glusterd2-cluster to allow smart volume creation to work. At the moment it works only for the Vagrant provisioned environment.

TODO: Make more universal
TODO: Convert to role

### vagrant-playbook.yml

This playbook combines all the above to provision the local cluster brought up by Vagrant


## How to use

### Requirements

- ansible
- python-virtualenv

#### Optional

- vagrant, vagrant-libvirt - Only required if you want to bring up the vagrant powered local cluster
- kubectl - Only required if you plan to manage the kube cluster from the ansible host

### External kube cluster

TODO: Write up how to use the deploy-gcs.yml playbook against an external K8S cluster.

### Local cluster using Vagrant

The provided Vagrantfile brings up a 3-node kubernetes cluster, with 3 1000GB
virtual disks attached to each node.

> Note that the virtual disks are created in the default storage pool of libvirt,
> though they are all sparse images.  The default storage pool is set to
> /var/lib/libvirt/images, so if used the cluster is used heavily for GCS
> testing, it is possible to run out of space on your root partition.

- Run the `prepare.sh` script to perform perliminary checks and prepare the virtualenv.

```
$ ./prepare.sh
```

- Activate the virtualenv
```
$ source gcs-env/bin/activate
(gcs-env) $
```

- Bring up the cluster. This will take a long time.

```
(gcs-env) $ vagrant up
```

Once begin using the cluster once its up by either,
- SSHing into one of the Kube nodes and running the `kubectl` commands in there

```
(gcs-env) $ vagrant ssh kube1
```

#### Resetting the cluster

If the Vagrant vms are restarted (say because of host reboot), the kubernete cluster cannot come back up. In such a case, reset the cluster and re-deploy kube and GCS.

```
(gcs-env) $ ansible-playbook --become kubespray/reset.yml
.
.
.
(gcs-env) $ ansible-playbook --skip-tags predeploy --become vagrant-playbook.yml
.
.
.
```

This will create a brand new cluster.

> Note that this currently does not work as the block devices that were added
> to the glusterd2-cluster would not be reset. This leaves behind stale LVM VGs
> causing new device adds to fail. For the time being destroy the local cluster
> completely `vagrant destroy -f` and start with a fresh `vagrant up`.
