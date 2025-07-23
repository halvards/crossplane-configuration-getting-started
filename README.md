# crossplane-configuration-getting-started

An introductory example to Crossplane and Compositions using provider-nop. This will enable provisioning of several different fake resource types.

This repository contains a reference configuration for [Crossplane](https://crossplane.io). This configuration is built with [provider-nop](https://marketplace.upbound.io/providers/crossplane-contrib/provider-nop), a Crossplane provider that simulates the creation of external resources.


## Overview

This platform offers APIs for setting up a variety of basic resources that mirror what you'd find in a Cloud Service Provider such as AWS, Azure, or GCP. The resource types include:

* [Cluster](apis/primitives/XCluster/), a resource that loosely represents a Kubernetes cluster.
* [NodePool](apis/primitives/XNodePool/), a resource that loosely represents a Nodepool in a Kubernetes cluster.
* [Database](apis/primitives/XDatabase/), a resource that loosely represents a cloud database.
* [Network](apis/primitives/XNetwork/), a resource that loosely represents a cloud network resource.
* [Subnetwork](apis/primitives/XSubnetwork/), a resource that loosely represents a subnetwork resource within a cloud network.
* [Service Account](apis/primitives/XServiceAccount/), a resource that loosely represents a service account in the cloud.

This configuration also demonstrates the power of Crossplane to build abstractions called "compositions", which assemble multiple basic resources into a more complex resource. These are demonstrated with:

* [CompositeCluster](apis/composition-basics/XCompositeCluster/), a resource abstraction that composes a cluster, nodepool, network, subnetwork, and service account.
* [AccountScaffold](apis/composition-basics/XAccountScaffold/), a resource abstraction that composes a service account, network, and subnetwork.

Learn more about Composite Resources in the [Crossplane
Docs](https://docs.crossplane.io/latest/concepts/compositions/).

## Quickstart

### Prerequisites

This guide deployes to a local Kubernetes cluster using [`kind`](https://kind.sigs.k8s.io/).

To run a `kind` cluster, you must install either Docker Desktop or Podman.

### Install tools

Install the required CLI tools:

```console
brew install crossplane helm kind kubernetes-cli
```

You can optionally also install `k9s` for a terminal-based dashboard.

### Create a Kubernetes cluster

Use [`kind`](https://kind.sigs.k8s.io/) to create a local Kubernetes cluster:

```console
kind create cluster --name crossplane-getting-started
```

### Install the Crossplane controller

Use Helm:

```console
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace \
  --version 1.20.0

kubectl rollout status --namespace crossplane-system deployment/crossplane
kubectl rollout status --namespace crossplane-system deployment/crossplane-rbac-manager
```

Reference: [Install Crossplane](https://docs.crossplane.io/latest/software/install/)

### Install the Getting Started configuration

Now you can install this reference platform. It's packaged as a [Crossplane
configuration package](https://docs.crossplane.io/latest/concepts/packages/)
so there is a single command to install it:

```console
crossplane xpkg install configuration xpkg.upbound.io/upbound/configuration-getting-started:v0.3.0
sleep 5
kubectl wait --for condition=Healthy provider.pkg.crossplane.io/crossplane-contrib-provider-nop
kubectl wait --for condition=Healthy configuration.pkg.crossplane.io/upbound-configuration-getting-started
```

Validate the installation by inspecting the provider and configuration packages:

```console
kubectl get providers,providerrevision

kubectl get configurations,configurationrevisions
```

Check the
[marketplace](https://marketplace.upbound.io/configurations/upbound/configuration-getting-started/)
for the latest version of this configuration package.

## Create composite resources using claims

You can now use the managed control plane to request resources which will simulate getting
provisioned in an external cloud service. You do this by creating
[claims](https://docs.crossplane.io/latest/concepts/claims/) against the APIs available on your
control plane. In our example here we simply create the claims directly.

Create a namespace to hold our claims:

```console
kubectl create namespace my-resources
```

Create a custom defined cluster:

```console
kubectl apply --filename examples/XCluster/claim.yaml --namespace my-resources

kubectl wait --for condition=Ready cluster.platform.acme.co/cluster1 --namespace my-resources
```

View the resource hierarchy created from the cluster claim:

```console
crossplane beta trace cluster.platform.acme.co/cluster1 --namespace my-resources
```

Create a custom defined database:

```console
kubectl apply --filename examples/XDatabase/claim.yaml --namespace my-resources

kubectl wait --for condition=Ready database.platform.acme.co/database1 --namespace my-resources
```

View the resource hierarchy created from the database claim:

```console
crossplane beta trace database.platform.acme.co/database1 --namespace my-resources
```

List the claims in the namespace we created:

```console
kubectl get claim,composite,managed --namespace my-resources
```

List the composite resources and managed resources in the cluster:

```console
kubectl get composite,managed
```

Explore additional resources in the [`examples`](./examples/) directory.

## Delete resources

To delete the provisioned resources, delete the claims:

```console
kubectl delete cluster.platform.acme.co/cluster1 --namespace my-resources

kubectl delete database.platform.acme.co/database1 --namespace my-resources
```

## Clean up

To uninstall the provider platform configuration, but keep the `kind` cluster:

```console
kubectl delete configurations.pkg.crossplane.io upbound-configuration-getting-started
```

Delete the `kind` Kubernetes cluster:

```console
kind delete cluster --name crossplane-getting-started
```
