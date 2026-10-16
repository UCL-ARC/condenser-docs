---
title: Deploying with kubectl
---

# Deploying with kubectl

Resources on Condenser are typically deployed through Harvester onto an
underlying kubernetes cluster. However, the cluster can be accessed directly using
the kubectl tool. This can be used to access features that are not yet enabled by
Harvester.

On your local computer, install [kubectl](https://kubernetes.io/docs/reference/kubectl/)
and obtain a [kubeconfig file](../troubleshooting/kubeconfig.md) for a cluster on
Condenser.

Export the location of the kubeconfig file and try the following command to see
the namespaces that you have permission to read:

``` sh
export KUBECONFIG=/path/to/kubeconfig.yaml
kubectl get ns
```

The output should be a similar table:

``` text
NAME                                     STATUS   AGE
arc-bespoke-ns                           Active   93d
arc-general-ns                           Active   204d
```

You can also use `kubectl apply` to deploy resources.

Kubernetes deployments are described by yaml files. The following files describe
a deployment like that created in the [Rancher GUI](./deploying_rancher.md)
and [Terraform](./deploying_terraform.md) tutorials. In those tutorials, Harvester
automatically deployed a volume to serve as the root block device for the VM. Here
we will need to deploy it manually.

## Describing the root block device

`pvc.yaml`

``` yaml hl_lines="4 5 11 12"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-kubectl-rootdisk
  namespace: my-ns
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 30Gi
  storageClassName: <INSERT LONGHORN STORAGECLASS>
  volumeMode: Block
```

The highlighted lines indicate where you may need to alter the deployment.
In particular, you need to use the storageClassName that corresponds to the VM image
you want to use. You can discover this by first listing the images available to you:

``` sh
$ kubectl get virtualmachineimage -A
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAMESPACE             NAME          DISPLAY-NAME                                SIZE         AGE
harvester-public      image-mb9nd   almalinux-9.4-20240507                      602603520    107d
```

Then describe the image that you want to use and find the name of the corresponding
storageClass:

``` sh hl_lines="26"
$ kubectl describe virtualmachineimage --selector='harvesterhci.io/imageDisplayName=almalinux-9.4-20240507' -A
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
Name:         image-mb9nd
Namespace:    harvester-public
Labels:       harvesterhci.io/image-type=raw_qcow2
              harvesterhci.io/imageDisplayName=almalinux-9.4-20240507
              harvesterhci.io/os-type=linux
Annotations:  harvesterhci.io/storageClassName: harvester-longhorn
API Version:  harvesterhci.io/v1beta1
Kind:         VirtualMachineImage
...
Status:
  Applied URL:  https://repo.almalinux.org/almalinux/9/cloud/x86_64/images/AlmaLinux-9-GenericCloud-9.4-20240507.x86_64.qcow2
  Conditions:
    Last Update Time:  2024-11-04T18:17:02Z
    Reason:            Imported
    Status:            True
    Type:              Imported
    Last Update Time:  2024-11-04T18:16:44Z
    Reason:            Initialized
    Status:            True
    Type:              Initialized
  Failed:              0
  Progress:            100
  Size:                602603520
  Storage Class Name:  longhorn-image-mb9nd
Events:                <none>
```

Or, like so:

``` sh
$ kubectl get virtualmachineimage --selector='harvesterhci.io/imageDisplayName=almalinux-9.4-20240507' -A -o json | jq '.items[0].status.storageClassName'
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
"longhorn-image-mb9nd"
```

## Describing the VM

`vm.yaml`

``` yaml hl_lines="4 5 12 32 33 35 36 37 40 45 54"
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: demo-kubectl
  namespace: my-ns
spec:
  runStrategy: RerunOnFailure
  template:
    spec:
      domain:
        cpu:
          cores: 2
        devices:
          disks:
            - bootOrder: 1
              disk:
                bus: virtio
              name: rootdisk
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - bridge: {}
              model: virtio
              name: nic-1
        machine:
          type: q35
        memory:
          guest: 16284Mi
        resources:
          limits:
            cpu: '2'
            memory: 16Gi
          requests:
            cpu: 125m
            memory: 10922Mi
      hostname: demo-kubectl
      networks:
        - multus:
            networkName: my-ns/my-network
          name: nic-1
      volumes:
        - name: rootdisk
          persistentVolumeClaim:
            claimName: demo-kubectl-rootdisk
        - cloudInitNoCloud:
            userData: |
              #cloud-config
              bootcmd:
                - [ dnf, config-manager, --set-enabled, crb ]
                - [ dnf, install, -y, epel-release ]

              ssh_authorized_keys:
                - <INSERT SSH PUBLIC KEY DATA HERE>
          name: cloudinitdisk
```

The highlighted lines indicate where you may need to alter the deployment.
In particular, you need to add your ssh public key to the cloudinit data. The claimName
field needs to correspond to the name of the PersistentVolumeClaim that you use to
provide the root block device.

## Deploying

Create the files described above (`pvc.yaml` and `vm.yaml`).
Then deploy these resources as follows:

``` sh
kubectl apply -f pvc.yaml
kubectl apply -f vm.yaml
```

Wait about 2 minutes for the IP address to become available:

``` sh
$ kubectl get virtualmachineinstance --namespace my-ns
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME           AGE    PHASE     IP            NODENAME          READY
demo-kubectl   58m    Running   10.134.X.X    harvester-2rbw7   True
```

Then you can [log in](../ssh_login.md), etc.

## Destroying

First delete the VM, then the pvc:

``` sh
kubectl delete -f vm.yaml
kubectl delete -f pvc.yaml
```
