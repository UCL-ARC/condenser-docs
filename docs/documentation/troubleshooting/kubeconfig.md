---
title: Obtaining a kubeconfig file
---

# Obtaining a kubeconfig file

A kubeconfig file is required to remotely deploy infrastructure on the Harvester
clusters within Condenser. You may need to provide it to `kubectl` or to the Harvester
Terraform provider. With a kubeconfig file you can take any action that your user
account is permitted to do, including the destruction of resources.

!!! note
    The kubeconfig file contains a secret token that uses your credentials to authenticate
    to the Harvester cluster. Do not share it with anyone. If your kubeconfig file
    is compromised, you can revoke the key from the [Account and API Keys](https://rancher.condenser.arc.ucl.ac.uk/dashboard/account)
    page in Rancher.

1. Log in to the [Rancher GUI](https://rancher.condenser.arc.ucl.ac.uk/)
2. In the left menu, click **Virtualization Management**
3. Tick the box next to a cluster from the **Harvester Clusters** list that you
   want to access
4. Click on **Download KubeConfig** at the top of the list. Your browser will download
   a file named `<cluster name>.yaml`

This is your kubeconfig file.

!!! note
    You can download one kubeconfig file that contains your credentials for multiple
    clusters with this method. The clusters are identified in the kubeconfig file with
    contexts. You can use the `--context` flag with `kubectl` to specify the cluster
    you want to use, and you can edit the kubeconfig file to change the default context.
    You can also configure a context for the Harvester Terraform provider.

The [Harvester Terraform provider](https://registry.terraform.io/providers/harvester/harvester/latest/docs#schema)
accepts the path to the kubeconfig file from the `KUBECONFIG` environment variable
or as configuration to the provider block.

``` hcl
provider "harvester" {
    kubeconfig = "/path/to/kubeconfig.yaml"
}
```

See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
for instructions on using a kubeconfig file with `kubectl`.
