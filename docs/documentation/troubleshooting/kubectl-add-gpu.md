---
title: Adding a GPU to a host VM manually
---

# Adding a GPU to a host VM manually

It is possible to assign a GPU to your virtual machine by editing the VM to add
a `hostDevices` section to the VM's YAML configuration. You can do this using `kubectl`:

```sh
kubectl -n arc-gpu-test-ns get vm
NAME             AGE   STATUS    READY
gpu-test-iac-0   32d   Running   True
gpu-test-iac-1   32d   Running   True

kubectl -n arc-gpu-test-ns edit vm gpu-test-iac-0
```

Or by editing the YAML configuration of your VM in the Rancher GUI.

Add the following to the devices section of the YAML configuration (at the level
of the other devices in this section):

```yaml hl_lines="8 9"
devices:
  disks:
  - bootOrder: 1
    disk:
      bus: virtio
    name: disk-0
  hostDevices:
  - deviceName: nvidia.com/GA100_A100_PCIE_80GB # Type of your device (via pcidevice)
    name: sl-g02-08-000031000                   # Name of your device
  interfaces:
  - bridge: {}
```

In order to get the `deviceName` above, you can use `kubectl` to find the resourceName of the pcidevice resource that represents the GPU card:

```sh
kubectl get pcidevice sl-g02-08-000031000 -o jsonpath='{.status.resourceName}'
nvidia.com/GA100_A100_PCIE_80GB
```

Once this configuration is saved, you will need to restart the VMI for this VM so that these changes
can be picked up. If your VMI was not running on the same node that the GPU is attached
to, it will be migrated automatically when you restart it.
