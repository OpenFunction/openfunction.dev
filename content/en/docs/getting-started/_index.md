---
title: "Getting started"
linkTitle: "Getting started"
weight: 5
description: >	
    Quick start with OpenFunction, including installation, demonstration of simple cases
---

## Overview

**OpenFunction** is a cloud-native, open source FaaS (function as a service) framework designed to let developers focus on their development intentions without worrying about the underlying runtime environment and infrastructure. Users can generate event-driven, dynamically scaling Serverless workloads by simply submitting a piece of code.

OpenFunction Features:

- Cloud-native, open source
- Automatically build code into OCI-compliant images
- Automatically deploys application with dynamic scaling capabilities
- Provides events framework to make functions event-driven
- Provides function version control and ingress traffic management capabilities

OpenFunction has two main capabilities:

- Serverless framework with CustomResourceDefinitions (CRD) [Function]({{<ref "../concepts/function">}}), [Builder]({{<ref "../concepts/builder">}}), [Serving]({{<ref "../concepts/serving">}})
- Events framework with CustomResourceDefinitions (CRD) [EventSource]({{<ref "../concepts/eventsource">}}), [EventBus (ClusterEventBus)]({{<ref "../concepts/eventbus">}}), [Trigger]({{<ref "../concepts/trigger">}})

## Compatibility

### Kubernetes compatibility matrix

The following versions are supported and work as we test against these versions in their respective branches. But note that other versions might work!

| OpenFunction                                                 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 |
| ------------------------------------------------------------ | --------------- | --------------- | --------------- | --------------- |
| [`release-0.3`](https://github.com/OpenFunction/OpenFunction/tree/v0.3.0) | &radic;         | &radic;         | &radic;         | &radic;         |
| [`release-0.4`](https://github.com/OpenFunction/OpenFunction/tree/v0.4.0) | &times;         | &radic;         | &radic;         | &radic;         |
| [`HEAD`](https://github.com/OpenFunction/OpenFunction/tree/main) | &times;         | &radic;         | &radic;         | &radic;         |

## Prerequisites

The current version of OpenFunction requires that you have a Kubernetes cluster with version ``>=1.18.6``.

In addition, you need to deploy several dependencies for the OpenFunction ```Builder``` and ```Serving```.

You can use the following command to setup OpenFunction ```Builder``` and ```Serving```:

```shell
sh hack/deploy.sh --all
```

This command will install dependencies of all supported ```Builder``` and ```Serving``` to your cluster.

You can also customize the installation with the following parameters:

| Parameter             | Comment                                                      |
| --------------------- | ------------------------------------------------------------ |
| -\-all                | Install all supported ```Builder``` and ```Serving```        |
| -\-with-shipwright    | Install Shipwright builder                                   |
| -\-with-knative       | Install Knative serving runtime                              |
| -\-with-openFuncAsync | Install OpenFuncAsync serving runtime                        |
| -\-poor-network       | Use this if you have poor network connectivity to GitHub/Googleapis |

## Install

You can install the OpenFunction platform by the following command:

- Install the latest stable version

```shell
kubectl create -f https://github.com/OpenFunction/OpenFunction/releases/download/v0.4.0/bundle.yaml
```

- Install the development version

```shell
kubectl create -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/main/config/bundle.yaml
```

> Note: When using non-default namespaces, make sure that the ClusterRoleBinding in the namespace is adapted.

### Sample: Run a function.

If you have already installed the OpenFunction platform, refer to [OpenFunction samples](https://github.com/OpenFunction/samples) to run a sample function.

## Uninstall 

- Uninstall the latest stable version

```shell
kubectl delete -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/release-0.4/config/bundle.yaml
```

- Uninstall the development version

```shell
kubectl delete -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/main/config/bundle.yaml
```
