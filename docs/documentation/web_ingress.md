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

!!! note
    If you are using a version of the Harvester Terraform Provider prior to 1.7.0,
    labels are not configurable for the [harvester_virtualmachine resource](https://registry.terraform.io/providers/harvester/harvester/1.7.0/docs/resources/virtualmachine).
    We recommend that you use a recent version of the provider so that the labels
    feature is available to you. However, if you are required to use an older version
    of the provider you can use tags instead with the following format:

    ``` yaml
    condenser_ingress_[site-key]_[label-name]: value
    ```

    For example, if you choose a key, `test`, the `hostname` label would be configured
using:

    ``` yaml
    condenser_ingress_test_hostname: some-hostname-here
    ```

## Configuration

### Using Terraform

If using the [Harvester Terraform provider](https://registry.terraform.io/providers/harvester/harvester/latest/docs),
ingress rules should be configured using the `tags` argument on the [`harvester_virtualmachine`](https://registry.terraform.io/providers/harvester/harvester/latest/docs/resources/virtualmachine)
resource.

#### Enable Ingress to a VM

Add the following tag to enable ingress to a VM:

``` yaml
condenser_ingress_isEnabled: true
```

#### Configure a Site

Each VM can support multiple sites - choose a unique key per site to ensure configuration
is applied to the correct site. Keys must be unique within a VM. You should add
a tag in the following format:

``` yaml
condenser_ingress_[site-key]_[label-name]: value
```

For example, if you choose a key, `test`, the `hostname` label would be configured
using:

``` yaml
condenser_ingress_test_hostname: some-hostname-here
```

#### Required Labels

- `condenser_ingress_[site-key]/hostname: [hostname]`: Used to determine the FQDN.
  The final ingressed FQDN will be `[hostname].[rancher project name].condenser.arc.ucl.ac.uk`

#### Optional Labels

- `condenser_ingress_[site-key]/port: [port]`: Target port (default 443 if `protocol`
  is https, 80 otherwise)
- `condenser_ingress_[site-key]/protocol: [protocol]`: Target protocol (default http)
- `condenser_ingress_[site-key]/vip: [vip]`: Target VIP (if the IP address is not
  assigned to the VM)
- `condenser_ingress_[site-key]/interface: [interface]`: Which network interface
  to use if the VM has multiple network interfaces (default `eth0`)

#### Advanced Nginx Configuration

In addition to basic ingress rules, all [nginx annotations](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)
are supported.

An annotation can be added to an ingress rule by substituting `nginx.ingress.kubernetes.io`
with `condenser_ingress_[site-key]_nginx`. For example, to annotate an ingress rule,
`test`, with `nginx.ingress.kubernetes.io/proxy-body-size: 8m`, add the following
tag to your VM:

- `condenser_ingress_test_nginx/proxy-body-size: 8m`

### Rancher GUI

To configure HTTPS ingress using the Rancher GUI, choose `Edit Config` on your VM
and navigate to `Instance Labels`.

**Note**: When saving your VM, Rancher will ask if you wish to restart the VM.
Restarting the VM is *not* necessary to configure ingress.

#### Enable Ingress to a VM

To enable a VM for ingress, add the Instance Label:

- `condenser.ingress/isEnabled: true`

#### Configure a Site

Each VM can support multiple sites - choose a unique key per site to ensure configuration
is applied to the correct site. Keys must be unique within a VM. You should add
a tag in the following format:

- `condenser.ingress.[site-key]/[label-name]: value`

For example, if you choose a key, `test`, the `hostname` label would be configured
using:

- `condenser.ingress.test/hostname: some-hostname-here`

#### Required Labels

- `condenser.ingress.[site-key]/hostname: <hostname>`: where the final ingressed
  FQDN is `<hostname>.<project name>.condenser.arc.ucl.ac.uk`

#### Optional Labels

- `condenser.ingress.[site-key]/port: [port]`: Target port (default 443 if protocol
  is https, 80 otherwise)
- `condenser.ingress.[site-key]/protocol: [protocol]`: Target protocol (default http)
- `condenser.ingress.[site-key]/vip: [vip]`: Target VIP (if the IP address is not
  assigned to the VM)
- `condenser.ingress.[site-key]/interface: [interface]`: Which network interface
  to use if the VM has multiple network interfaces (default `eth0`)

#### Advanced Configuration

In addition to basic ingress rules, all [nginx annotations](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)
are supported.

An annotation can be added to an ingress rule by substituting `nginx.ingress.kubernetes.io`
with `condenser.ingress.[site-key].nginx`. For example, to annotate an ingress rule,
`test`, with `nginx.ingress.kubernetes.io/proxy-body-size: 8m`, add the following
instance label to your VM:

- `condenser.ingress.test.nginx/proxy-body-size: 8m`

### Examples

#### Basic Ingress

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80:

- `condenser.ingress/isEnabled: true`
- `condenser.ingress.test/hostname: test-host`

#### Basic Ingress with Terraform

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80:

```hcl
  tags = {
    condenser_ingress_isEnabled = true
    condenser_ingress_test_hostname = "test-host"
  }
```

#### Advanced Ingress with Terraform

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 80 with `proxy-body-size` set to 8m

```hcl
  tags = {
    condenser_ingress_isEnabled = true
    condenser_ingress_test_hostname = "test-host"
    condenser_ingress_test_nginx_proxy-body-size = "8m"
  }
```

#### HTTPS Ingress

Create an ingress, `test`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to the VM on port 443 using HTTPS:

- `condenser.ingress/isEnabled: true`
- `condenser.ingress.test/hostname: test-host`
- `condenser.ingress.test/port: 443`
- `condenser.ingress.test/protocol: https`

#### Ingress to a K3s VIP on a custom port

Create an ingress, `testvip`, which proxies `test-host.<project name>.condenser.arc.ucl.ac.uk`
to a K3s cluster's VIP, 10.134.8.9 on port 8080 using HTTP:

- `condenser.ingress/isEnabled: true`
- `condenser.ingress.testvip/hostname: test-host`
- `condenser.ingress.testvip/port: 8080`
- `condenser.ingress.testvip/vip: 10.134.8.9`

#### Multiple Ingresses

Create two ingresses, `testone` and `testtwo`, which proxy `testone.<project name>.condenser.arc.ucl.ac.uk`
and `testtwo.<project name>.condenser.arc.ucl.ac.uk` to the VM on port 8080/8081
respectively using HTTP:

- `condenser.ingress/isEnabled: true`
- `condenser.ingress.testone/hostname: testone`
- `condenser.ingress.testone/port: 8080`
- `condenser.ingress.testtwo/hostname: testtwo`
- `condenser.ingress.testtwo/port: 8081`

#### Multiple Ingresses with advanced configuration

Create two ingresses, `testone` and `testtwo`, which proxy `testone.<project name>.condenser.arc.ucl.ac.uk`
and `testtwo.<project name>.condenser.arc.ucl.ac.uk` to the VM on port 8080/8081
respectively using HTTP. `testone` requires a `proxy-buffer-size` of 8k, whilst
`testtwo` needs a `proxy-body-size` of 8m:

- `condenser.ingress/isEnabled: true`
- `condenser.ingress.testone/hostname: testone`
- `condenser.ingress.testone/port: 8080`
- `condenser.ingress.testone.nginx/proxy-buffer-size: 8k`
- `condenser.ingress.testtwo/hostname: testtwo`
- `condenser.ingress.testtwo/port: 8081`
- `condenser.ingress.testtwo.nginx/proxy-body-size: 8m`
