---
title: Deploying with Terraform
---

# Deploying with Terraform

You can use the [Terraform](https://developer.hashicorp.com/terraform) configuration
language to automatically create and destroy virtual resources on Condenser. Hashicorp,
the developer of Terraform, has produced extensive documentation and many [tutorials](https://developer.hashicorp.com/terraform/tutorials)
for writing Terraform modules.

Here we have created a short tutorial for writing and deploying a simple Terraform
module that will use the Harvester Terraform [provider](https://developer.hashicorp.com/terraform/language/providers)
to launch a small VM on Condenser. The VM is optionally configured for SSH access.

## Preparation

1. Enable the UCL VPN
2. Install the [`terraform`](https://developer.hashicorp.com/terraform/install?product_intent=terraform)
and [`kubectl`](https://kubernetes.io/releases/download/#kubectl) command line tools
on your local computer
3. Create the [`kubeconfig.yaml` file](../troubleshooting/kubeconfig.md)
4. Using the Rancher GUI, find the name of a namespace in which you have permission
to deploy resources
5. If you configure the VM for SSH access, you will also need:
    1. The name of a VLAN network in the namespace
    2. The SSH public key data for your SSH key pair

## Write a basic Terraform module to launch a VM

You may wish to start from the [UCL-ARC/terraform-template](https://github.com/UCL-ARC/terraform-template),
which provides a suggested file structure and a set of useful GitHub actions for
Terraform projects.

If you want to use the template, go to the [repository page](https://github.com/UCL-ARC/terraform-template).
In the upper right, click on "Use this template" and select "Create a new repository".
After you have created the repository, clone it onto your local computer. In your
terminal, navigate to the `terraform-template` directory. This will be the root
directory of your terraform module.

If you use the template, first delete the template content in each of the `*.tf`
files.

Alternatively, on your local computer create a new directory. This will be the
root directory for your terraform module. In the new directory, create the following
empty files:

``` text
main.tf
output.tf
variables.tf
versions.tf
```

Copy the following content into the `versions.tf` file:

``` hcl
terraform {
  required_providers {
    harvester = {
      source  = "harvester/harvester"
      version = "0.6.6"
    }
  }

  required_version = ">= 1.8.5"
}

provider "harvester" {}
```

Copy the following content into the `main.tf` file:

``` hcl hl_lines="2 3 7 9 10 12 13 25"
data "harvester_image" "img" {
  display_name = "almalinux-9.4-20240805"
  namespace    = "harvester-public"
}

resource "harvester_virtualmachine" "vm" {
  name        = "demo"
  namespace   = var.namespace
  description = "Demo VM"
  hostname    = "demo"

  cpu    = 2
  memory = "8Gi"

  restart_after_update = true
  efi                  = true
  secure_boot          = true
  run_strategy         = "RerunOnFailure"
  reserved_memory      = "100Mi"
  machine_type         = "q35"

  disk {
    name       = "rootdisk"
    type       = "disk"
    size       = "50Gi"
    bus        = "virtio"
    boot_order = 1

    image       = data.harvester_image.img.id
    auto_delete = true
  }

  network_interface {
    name           = "default"
  }
}
```

The highlighted lines indicate where you can change the base image and other aspects
of the VM.

Copy the following content into the `variables.tf` file:

``` hcl
variable "namespace" {
  type        = string
  description = "Namespace that the VM will be deployed in"
}
```

### Configure the VM for SSH access

These additions to the module are optional for deploying a VM, but are required to
configure the VM to be accessible by SSH.

Replace the `network_interface` block in the `harvester_virtualmachine.vm` resource
in `main.tf` with the following:

``` hcl
  network_interface {
    name           = "nic-1"
    wait_for_lease = true
    type           = "bridge"
    network_name   = var.network_name
  }
```

Add the following block to the `harvester_virtualmachine.vm` resource in `main.tf`:

``` hcl
  cloudinit {
    user_data = <<EOF
#cloud-config
package_update: true
packages:
  - qemu-guest-agent
runcmd:
  - - systemctl
    - enable
    - --now
    - qemu-guest-agent.service
ssh_authorized_keys:
  - ${var.ssh_public_key_data}
EOF
  }
```

Append the following content to the `variables.tf` file:

``` hcl
variable "network_name" {
  type        = string
  description = "Name of a network in the namespace"
}

variable "ssh_public_key_data" {
  type        = string
  description = "SSH public key data"
}
```

Copy the following content into the `output.tf` file:

``` hcl
output "ip_address" {
  value = harvester_virtualmachine.vm.network_interface[0].ip_address
}
```

## Deploy the Terraform module

Set the `KUBECONFIG` environment variable to point to your [kubeconfig file](../troubleshooting/kubeconfig.md):

``` sh
export KUBECONFIG="/path/to/kubeconfig.yaml"
```

From the root of your terraform module, run the following commands.

``` sh
terraform init
terraform apply
```

When prompted as below, enter the name of your namespace, and any other variables,
and then `yes` to launch the VM.

``` text
...
var.namespace
  Namespace that the VM will be deployed in

  Enter a value:
> my-ns
...
Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
> yes
```

!!! note
    See the [Terraform documentation](https://developer.hashicorp.com/terraform/language/values/variables#assigning-values-to-root-module-variables)
    to automatically configure input variables for your Terraform module.

Terraform will report the following when the VM is deployed:

``` text
...
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You can monitor VMs that you launch with Terraform with the Rancher GUI or with `kubectl`:

``` sh
kubectl get virtualmachineinstance.kubevirt.io --namespace my-ns
```

## Log in to the VM with SSH

If you configured the VM for SSH access, the module will issue an IP address as output.
Run `terraform output` or `terraform state show "harvester_virtualmachine.vm"` to
print it again.

For VMs created from an Almalinux image, the username will be `almalinux`.

Follow these instructions to [access the VM with SSH](../ssh_login.md).

## Destroy the VM

When you are finished, run `terraform destroy` and answer `yes` when prompted
to tear the VM down.

## Troubleshooting

### IP address is not reported by `kubectl` or the Rancher GUI

If the IP address is successfully reported by Terraform, then the address has been
assigned but the VM needs to be restarted before it will appear in the VM's attributes
in the kubernetes cluster. You can restart a VM using the [Rancher GUI](./deploying_rancher/#change-a-vms-state),
or by issuing the `reboot` command over SSH. You can configure the VM to reboot automatically
after it launches by adding the following schema to the [cloudinit `user_data`](https://cloudinit.readthedocs.io/en/latest/reference/modules.html#power-state-change):

``` yaml
power_state:
  mode: reboot
```

## Further reading

- [Terraform configuration language documentation](https://developer.hashicorp.com/terraform/language)
- [Harvester Terraform provider documentation](https://registry.terraform.io/providers/harvester/harvester/latest/docs)
