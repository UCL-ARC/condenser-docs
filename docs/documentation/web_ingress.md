---
title: Configuring a VM for web ingress
---

# Configuring a VM for web ingress

Web ingress to Condenser can be configured automatically by labeling resources.
Ingress must be enabled on your tenancy before this feature can be used.

HTTPS ingress will be configured with:

- A URL: `https://[hostname].[rancher project name].condenser.arc.ucl.ac.uk`
- A valid LetsEncrypt certificate

By default traffic will be routed to the `eth0` network interface on the VM, using
HTTP on port 80.

If a VM's IP address changes, the ingress rule will be updated. If a VM is powered
off, the ingress rule will be deleted. Once the VM is powered back on, the ingress
rule will be recreated.

## Configuration

### Adding Labels

#### Rancher GUI

To configure HTTPS ingress using the Rancher GUI, choose `Edit Config` on your VM
and navigate to `Instance Labels`.

!!! note
    When saving your VM, Rancher will ask if you wish to restart the VM.
    Restarting the VM is *not* necessary to configure ingress.

#### Terraform

In a Terraform module, you can set labels on a [`harvester_virtualmachine`](https://registry.terraform.io/providers/harvester/harvester/1.7.0/docs/resources/virtualmachine)
resource like so:

```hcl
labels = {
    "condenser.ingress/isEnabled" = true
  }
```

!!! note
    If you are using a version of the Harvester Terraform Provider prior to 1.7.0,
    labels are not configurable for the [harvester_virtualmachine resource](https://registry.terraform.io/providers/harvester/harvester/1.7.0/docs/resources/virtualmachine).
    We recommend that you use a recent version of the provider so that labels and
    other features are available to you. However, if you are required to use an
    older version of the provider you can use tags instead. Tags with the following
    format will be parsed into the correct labels by the ingress feature:

    ```hcl
    tags = {
      "condenser_ingress_[site-key]_[label-name]" = "value"
    }
    ```

    For example, if you choose a key, `test`, the `hostname` label would be configured
using:

    ```hcl
    tags = {
      "condenser_ingress_isEnabled" = true
      "condenser_ingress_test_hostname" = "some-hostname-here"
    }
    ```

    To add nginx annotations, use the `nginx` suffix and the annotation key:

    ```hcl
    tags = {
      ...

      "condenser_ingress_[site-key]_nginx/[annotation-key]" = "value"
    }
    ```

    For example,

    ```hcl
    tags = {
      ...

      "condenser_ingress_test_nginx/proxy-body-size" = "8m"
    }
    ```

### Enable Ingress to a VM

To enable a VM for ingress, add the Instance Label:

```hcl
labels = {
    "condenser.ingress/isEnabled" = true
  }
```

### Configure a Site

Each VM can support multiple sites - choose a unique key per site to ensure configuration
is applied to the correct site. Keys must be unique within a VM. You should add
a tag in the following format:

```hcl
labels = {
    "condenser.ingress.[site-key]/[label-name]: value
  }
```

For example, if you choose a key, `test`, the `hostname` label would be configured
using:

```hcl
labels = {
    "condenser.ingress/isEnabled" = true
    "condenser.ingress.test/hostname" = "some-hostname-here"
  }
```

### Required Labels

These labels are required:

```hcl
labels = {
    "condenser.ingress/isEnabled" = true
    "condenser.ingress.test/hostname" = "some-hostname-here"
  }
```

#### `isEnabled`

Labels the virtual machine so that an ingress will be generated.

#### `hostname`

The final ingressed FQDN is `<hostname>.<project name>.condenser.arc.ucl.ac.uk`.

### Optional Labels

These labels are optional:

```hcl
labels = {
    ...

    "condenser.ingress.[site-key]/port"      = [port]
    "condenser.ingress.[site-key]/protocol"  = [protocol]
    "condenser.ingress.[site-key]/interface" = [interface]
    "condenser.ingress.[site-key]/vip"       = [vip]
  }
```

#### `port`

Target port (default 443 if protocol is https, default 80 if protocal is otherwise)

#### `protocol`

Target protocol (default https if port is 443, default http if port is otherwise)

#### `interface`

Use this to select which network interface to use if the VM has multiple network
interfaces (default `eth0` if there are multiple interfaces present, otherwise will
use the single present interface)

#### `vip`

Target VIP, used to set the VIP if the IP address is not assigned to the VM (default
is the interface's IP)

### Advanced Configuration

In addition to basic ingress rules, all [nginx annotations](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)
are supported.

An annotation can be added to an ingress rule by substituting `nginx.ingress.kubernetes.io`
with `condenser.ingress.[site-key].nginx`. For example, to annotate an ingress rule,
`test`, with `nginx.ingress.kubernetes.io/proxy-body-size: 8m`, add the following
instance label to your VM:

```hcl
labels = {
    ...

    "condenser.ingress.test.nginx/proxy-body-size" = "8m"
  }
```

## Examples

### Basic Ingress

To create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80, add these labels to the VM configuration with the Rancher
GUI:

```yaml
condenser.ingress/isEnabled: true
condenser.ingress.test/hostname: test-host
```

### Basic Ingress with Terraform

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80, by labeling a `harvester_virtualmachine` resource like so:

```hcl
labels = {
    "condenser.ingress/isEnabled" = true
    "condenser.ingress.test/hostname" = "test-host"
  }
```

### Advanced Ingress with Terraform

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80 with `proxy-body-size` set to 8m

```hcl
labels = {
    "condenser.ingress/isEnabled"                  = true
    "condenser.ingress.test/hostname"              = "test-host"
    "condenser.ingress.test.nginx/proxy-body-size" = "8m"
  }
```

### HTTPS Ingress

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 443 using HTTPS:

```yaml
condenser.ingress/isEnabled: true
condenser.ingress.test/hostname: test-host
condenser.ingress.test/port: 443
condenser.ingress.test/protocol: https
```

### Ingress to a K3s VIP on a custom port

Create an ingress, `testvip`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to a K3s cluster's VIP, 10.134.8.9 on port 8080 using HTTP:

```yaml
condenser.ingress/isEnabled: true
condenser.ingress.testvip/hostname: test-host
condenser.ingress.testvip/port: 8080
condenser.ingress.testvip/vip: 10.134.8.9
```

### Multiple Ingresses

Create two ingresses, `testone` and `testtwo`, which proxy `testone.<project name>.condenser.arc.ucl.ac.uk`
and `testtwo.<project name>.condenser.arc.ucl.ac.uk` to the VM on port 8080/8081
respectively using HTTP:

```yaml
condenser.ingress/isEnabled: true
condenser.ingress.testone/hostname: testone
condenser.ingress.testone/port: 8080
condenser.ingress.testtwo/hostname: testtwo
condenser.ingress.testtwo/port: 8081
```

### Multiple Ingresses with advanced configuration

Create two ingresses, `testone` and `testtwo`, which proxy `testone.<project name>.condenser.arc.ucl.ac.uk`
and `testtwo.<project name>.condenser.arc.ucl.ac.uk` to the VM on port 8080/8081
respectively using HTTP. `testone` requires a `proxy-buffer-size` of 8k, whilst
`testtwo` needs a `proxy-body-size` of 8m:

```yaml
condenser.ingress/isEnabled: true
condenser.ingress.testone/hostname: testone
condenser.ingress.testone/port: 8080
condenser.ingress.testone.nginx/proxy-buffer-size: 8k
condenser.ingress.testtwo/hostname: testtwo
condenser.ingress.testtwo/port: 8081
condenser.ingress.testtwo.nginx/proxy-body-size: 8m
```
