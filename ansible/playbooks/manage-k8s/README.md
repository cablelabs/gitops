# Red Hat OpenShift Cluster Creation

Playbooks to create New OpenShift cluster and add worker nodes.

## Prepare Utility node

Installs required packages and readys a new utility node to use for cluster install.

Can be a VM, container, etc... This is just the location you are running the ansible playbooks from. Needs to be included in inventory as the utility node.

```bash
ansible-playbook -i inventories/{{ cluster}}/hosts --ask-vault-pass playbooks/manage-k8s/prepare_utility_host.yml
```

## Generate ISOs for install

All isos

```bash
ansible-playbook -i inventories/{{ cluster }}/hosts --ask-vault-pass playbooks/manage-k8s/generate_isos.yml
```

Single iso

```bash
ansible-playbook -i inventories/{{ cluster }}/hosts --ask-vault-pass playbooks/manage-k8s/generate_isos.yml --limit {{ node }}
```

## Initial cluster deployment

This playbooks will deploy a new cluster consisting of a temporary bootstrap node and three controller nodes</br>
The initial bootstrap node is typically the last worker in the cluster temporarily deployed as the bootstrap node.</br>

```bash
ansible-playbook -i inventories/{{ cluster }}/hosts --ask-vault-pass playbooks/manage-k8s/provision_new_cluster.yml
```

## Add worker nodes to cluster

This playbook adds a worker node to the cluster. 

```bash
ansible-playbook -i inventories/{{ cluster }}/hosts --ask-vault-pass playbooks/manage-k8s/add_node.yml --limit worker-0.{{ cluster }}.convergence.cablelabs.com
```

## Post Tasks

* Upload the kubeconfig file details to a secrets server. Default location for this file is {{ base_dir }}/{{ cluster_name }}-install/auth/kubeconfig
* Upload the kubeadmin-password password to a secrets server. Default location for this file is {{ base_dir }}/{{ cluster_name }}-install/auth/kubeadmin-password
* Verify cluster registration via OpenShift Cluster Manager

## How to encrypt a secret in ansible vault

```bash
ansible-vault encrypt_string --ask-vault-pass 'secret to encrypt' --name 'variable name for secret' >> inventories/{{ cluster }}/group_vars/all/vault_required/secrets.yml
```

Assumes running from the base ansible directory.


## Known Issues

* Occasionally the Boot Once from Virtual Media gets into a loop and boots multiple times from virtual media. You need to watch for this via the Console and if you see it happening go into the bios and disable the boot from virtual media and try again.
* If you need to add a worker node from a new utility node you'll need to manually populate the kubeconfig and kubeadmin-password files for usage in the playbook.
* Sometimes the hardware bios time will be too far off from the bootstrap-node time during cluster install. When this happens you'll want to manually log into the bios and update the time to the correct time.
* Automation hasn't been written to configure any specific hardware nic settings such as SRIOV at this time, any of those changes need to happen manually via the system BIOS.
* [CSR Issues](https://access.redhat.com/solutions/3716861)
* Intel Cascade boxes occasionally get a ghost entry in the BIOS that prohbits booting from the rhcos install iso. To fix this you need to boot a live CD and run efibootmgr and delete any Red Hat entries, example below.
```bash
efibootmgr # lists boot entries
efibootmgr -b {{ boot entry id }} -B
```
* On Intel boxes you will manually need to unmount the ISO so it doesn't get into a boot loop. You can use the GUI from the web interface or SDPTool. Monitor from the console and when it reboots in Install unmount the ISO.
```bash
sudo /bin/SDPTool {{ BMC IP }} {{ BMC USER }} {{ BMC PASSWORD }} unmount
```

## Troubleshooting

* One of the potential issues you can encounter with bootstrap node is it not getting an assigned IP. In such a case you can do the following steps.

** Manually try to update the networking with the following

```bash
ip addr add <IP>/24 dev <interface>
ip r add default via <IP> dev <interface>
```

* Another potential failure on bootstrap node can be that the coreos-installer service fails.

** You can go into maintenance mode by pressing Enter and run the below command to manually start it

```bash
systemctl start coreos-installer.service

```bash
* On a coreos-installer service failure you can keep trying to continue the install process multiple times by pressing CTRL+ D until the coreos-installer service starts.

## Version history

* 0.1 Initial Release
* 0.2 Updated to allow different versions of OCP to be installed from the same utility node.

## Acknowledgments
* https://gitlab.signal9.gg/aaustin/ocp-hybrid-ansible-share
