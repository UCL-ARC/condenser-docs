---
title: About the platform
---

# About the Platform

Condenser is a private cloud platform for research computing at UCL. It provides
a cost-effective, private alternative to public cloud platforms. The platform is
developed and maintained by the [Advanced Research Computing Centre](https://www.ucl.ac.uk/advanced-research-computing/).
All hardware resides in UCL facilities.

## Clusters

Condenser comprises the following clusters:

Cluster | Hardware | Description
------- | -------- | ---
sl-p02  | Intel CPU | Workloads that only require CPUs
sl-g01  | Intel CPU + Nvidia GPU | Mixed GPU-CPU workloads
sl-g02  | Intel CPU + Nvidia GPU | Mixed GPU-CPU workloads
sl-g03  | Intel CPU + Nvidia GPU | VAT-exempt, mixed GPU-CPU workloads

## Tenancy

Access to Condenser is organised according to tenancies. Users in a tenancy are
provided with access to a [Kubernetes](https://kubernetes.io/docs/home/) cluster.
Users can deploy virtual machines and related resources with [Harvester](https://docs.harvesterhci.io).
[Rancher](https://rancher.condenser.arc.ucl.ac.uk/dashboard/) is used to provide
access to the cluster.

A tenant consists of one or more projects, with at most one project on each cluster.
Each project is provided with one or more namespaces to isolate their resources from
neighbor tenants on the same cluster. Each namespace is also provided with an isolated
network. Resource quotas (CPU, RAM, and data storage) are applied to the namespace,
rather than to individual users. Most tenants will have one namespace on one cluster,
as shown in the diagram below.

![Tenancy diagram](assets/condenser-tenancy.svg)

## Sensitive data

Condenser is not certified for the secure storage and processing
of [sensitive data](https://www.ucl.ac.uk/library/open-science-research-support/research-data-management/best-practices/how-guides/handling-sensitive).
However, it is possible to build a secure platform on Condenser. When planning projects
with sensitive data on Condenser, please first discuss your requirements with the
[Information Governance Advisory Service](https://sensitive%20research%20data/).
Consult with us during onboarding, and plan to collaborate with a Research Data Engineer.
