---
title: Frequently asked questions
---

# Frequently Asked Questions

## How can I access my resources on Condenser with `kubectl` or Terraform?

Typically you will need to use your [kubeconfig file](./kubeconfig.md). If you have
access to projects on multiple clusters, take care to use the correct cluster context.
For example, if your kubeconfig file contains tokens for both `sl-p02` and `sl-g01`,
you can use the `context` option to specify which cluster you want to use:

``` sh
kubectl --context sl-p02 get vm -n my-namespace-ns
```

## How can I attach a GPU to my VM?

Configure `kubectl` with your [kubeconfig](./kubeconfig.md) file, then follow [these
instructions](./kubectl-add-gpu.md) to attach your GPU card to a VM by PCI passthrough.
Contact the [Environments team](https://io.uk.xurrent.com/cnQ9NTI4MA) if you need
the name of your PCI device.

Currently the Harvester GUI is not enabled to attach GPUs to VMs.

## How can I make changes to my tenant?

To make any of the following changes, submit a request on MyServices using the
[Tenancy change or termination request](https://io.uk.xurrent.com/cnQ9NTI0Mg) form:

- Change compute or storage resource quota
- Enable or disable web ingress
- Change tenancy users
- Change tenancy owner
- Delete a project
- Terminate a tenancy

For anything not covered above, submit a [Tenancy support request](https://io.uk.xurrent.com/cnQ9NTI4MA).

## My VM is not working; how can I fix it?

First, examine your virtual machine in the Harvester GUI for error messages. The
"Events" tab on the left side of a virtual machine's page may also have useful information.

If there are no error messages in the Harvester GUI, there may be a problem that
is internal to the VM. Restart the virtual machine, then use the WebVNC console to
watch the VM boot up and note any errors that occur during the boot.
