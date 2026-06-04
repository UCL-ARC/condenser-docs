---
title: Configuring a VM for SSH access
---

# Configuring a VM for SSH access

To be accessible by SSH, a VM needs to be configured with a [VLAN Network](https://docs.harvesterhci.io/latest/networking/harvester-network/)
and the SSH public key data needs to be provided via [cloudinit](https://cloudinit.readthedocs.io/en/latest/reference/modules.html#ssh).
We have documentation demonstrating this for [Terraform deployments](./deploying_resources/deploying_terraform.md/#configure-the-vm-for-ssh-access)
and for the [Rancher GUI](./deploying_resources/deploying_rancher.md).

After the VM is configured, the IP address for the VM can be found in the Rancher
GUI or by using `kubectl` with a suitable [kubeconfig file](./troubleshooting/kubeconfig.md)
and the name of the namespace where the VM is deployed. This command can be used
to list a table of virtual machines and their IP addresses; replace `my-ns` with
the appropriate namespace.

``` sh
kubectl get virtualmachineinstance.kubevirt.io --namespace my-ns
```

!!! note
   Tenant networks provide IP addresses in the `10.134.X.X` range. If your VM has
   been assigned an address in a different range, it may not be using your tenant
   network.

The VM's base image will typically be configured with a default username with SSH
login enabled. The username varies depending on the base image of the VM.

Image OS  | Default SSH username
--------- | ---
RHEL      | `cloud-user`
Almalinux | `almalinux`
Ubuntu    | `ubuntu`
Debian    | `debian`

VMs are not typically configured to allow SSH login to the root account.

Follow these instructions to [access a VM with SSH](ssh_login.md).
