---
title: Deploying external storage
---

# Deploying external storage

!!!note
     The steps below expects that a tenancy has been allocated a resource quota for *ibm-storage-array*

     Please contact the Environments team for a resource quota.

The IBM FlashSystem 5200 is an NVMe storage option that provides Condenser with terabytes of additional storage.

## Confirm that a resource quota is available

1. Export your kubeconfig
    1. `export KUBECONFIG=/path/to/kubeconfig.yaml`
2. Query assigned quotas
    2. `kubectl get resourcequotas -n <your_namespace>`

The result will be similar to the snippet below. Check that *ibm-storage-array-quota* is present:

```
NAME                      REQUEST                                                                  LIMIT                                                      AGE
default-shbj9             requests.storage: 1927Gi/5000Gi                                          limits.cpu: 30195m/64, limits.memory: 131271080585/384Gi   245d
ibm-storage-array-quota   ibm-storage-array.storageclass.storage.k8s.io/requests.storage: 0/50Ti                                                              45h
```

## Create a new volume

A volume must be created on the virtual machine so that the storage can be formatted and consumed:

1. Select **Virtual Machines**
2. Select your virtual machine
3. Select **Show Configuration**
4. Select **Edit Config**
5. Select **Volumes**
6. Select **Add Volume**
7. Select **Storage Class**
8. Choose **ibm-storage-array**
9. Select **Save**

The virtual machine will reboot and the volume will be made available to the guest operating system.

## Format a new disk

To make the storage functional, the new volume must be formatted on the guest operating system:

```
# Find the new disk
lsblk

# Look for the disk that has no partitions and confirm its empty
sudo blkid /dev/vdb

# Create a filesystem
# XFS: For large files and high throughput
# ext4: For smaller files and general purpose tasks
sudo mkfs.ext4 /dev/vdb

# Mount it
sudo mkdir -p /data
sudo mount /dev/vdb /data
df -h /data

# Create persistence by copying the UUID and adding it to /etc/fstab
sudo blkid /dev/vdb

# Add the following line to /etc/fstab
UUID=<your-id> /data ext4 defaults,nofail 0 0

# Permissions
# The mount will be owned by root by default. Adjust the permissions as needed
sudo chown youruser:youruser /data
```



