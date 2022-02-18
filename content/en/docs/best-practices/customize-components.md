---
title: "Customize OpenFunction Component Versions"
linkTitle: "Customize OpenFunction Component Versions"
weight: 5100
description: >	
    Learn how to customize OpenFunction component versions.
---

This document describes how to customize OpenFunction component versions.

## Component Compatibility Matrix

OpenFunction relies on several components, including Knative Serving, Dapr, KEDA, Shipwright, and Tekton. Some of these components require a specific Kubernetes version. The OpenFunction CLI installs the default versions of components based on a specific Kubernetes version.

The following table describes the default compatibility matrix of OpenFunction components.

| Component              | Kubernetes 1.17 | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20+ | CLI Parameter    | Description                                                  |
| ---------------------- | --------------- | --------------- | --------------- | ---------------- | ---------------- | ------------------------------------------------------------ |
| Knative Serving        | 0.21.1          | 0.23.3          | 0.25.2          | 1.0.1            | `--knative`      | The runtime for synchronous functions.                       |
| Kourier                | 0.21.0          | 0.23.0          | 0.25.0          | 1.0.1            | `--knative`      | The default network layer for Knative.                       |
| Serving Default Domain | 0.21.0          | 0.23.0          | 0.25.0          | 1.0.1            | `--knative`      | The default DNS layout for Knative.                          |
| Dapr                   | 1.5.1           | 1.5.1           | 1.5.1           | 1.5.1            | `--async`        | The distributed application runtime for asynchronous functions. |
| KEDA                   | 2.4.0           | 2.4.0           | 2.4.0           | 2.4.0            | `--async`        | The autoscaler of asynchronous function runtime.             |
| Shipwright             | 0.6.1           | 0.6.1           | 0.6.1           | 0.6.1            | `--shipwright`   | The function build framework.                                |
| Tekton Pipelines       | 0.23.0          | 0.26.0          | 0.29.0          | 0.30.0           | `--shipwright`   | The function build pipeline.                                 |
| Cert Manager           | 1.5.4           | 1.5.4           | 1.5.4           | 1.5.4            | `--cert-manager` | OpenFunction webhook certificate manager (for OpenFunction v0.4.0+ only). |
| Ingress Nginx          | N/A             | N/A             | 1.1.0           | 1.1.0            | `--ingress`      | Function ingress controller (for OpenFunction v0.4.0+ only). |

## Customize Component Versions

The OpenFunction CLI keeps the details about installed component in `$home/.ofn/<cluster name>-inventory.yaml`. To make installation more flexible, the OpenFunction CLI supports using environment variables to customize component versions. The following table describes these environment variables.

| Environment Variable     | Description                                   | Example Value  |
| ------------------------ | --------------------------------------------- | -------------- |
| DAPR_VERSION             | The version of Dapr                           | 1.4.3, 1.5.1   |
| KEDA_VERSION             | The version of KEDA                           | 2.4.0, 2.5.0   |
| KNATIVE_SERVING_VERSION  | The version of Knative Serving                | 0.23.3, 1.0.1  |
| KOURIER_VERSION          | The version of Kourier                        | 0.23.0, 0.26.0 |
| DEFAULT_DOMAIN_VERSION   | The version of Serving Default Domain         | 0.23.0, 0.26.0 |
| SHIPWRIGHT_VERSION       | The version of Shipwright (under development) | 0.6.1          |
| TEKTON_PIPELINES_VERSION | The version of Tekton Pipelines               | 0.26.0, 0.29.0 |
| INGRESS_NGINX_VERSION    | The version of Ingress Nginx                  | 1.1.0          |
| CERT_MANAGER_VERSION     | The version of Cert Manager                   | 1.5.4          |

{{% alert title="Note" color="success" %}}

- Make sure the customized component versions are compatible with your Kubernetes version before using these environment variables.
- If you use environment variables when installing OpenFunction, make sure that you set the environment variables to the same value when uninstalling OpenFunction.
- When installing a component, the customized component information will be obtained in the following order: `YAML file environment variables > version environment variables > $home/.ofn/inventory.yaml`.

{{% /alert %}}

## Customize YAML File Paths of Components

The following table describes the environment variables in a YAML file for a component.

| Environment Variable      | Description                                          |
| ------------------------- | ---------------------------------------------------- |
| KEDA_YAML                 | The path of KEDA YAML file                           |
| KNATIVE_SERVING_CRD_YAML  | The path of Knative Serving CRDs YAML file           |
| KNATIVE_SERVING_CORE_YAML | The path of Knative Serving core YAML file           |
| KOURIER_YAML              | The path of Kourier YAML file                        |
| DEFAULT_DOMAIN_YAML       | The path of Serving Default Domain YAML file         |
| SHIPWRIGHT_YAML           | The path of Shipwright YAML file (under development) |
| TEKTON_PIPELINES_YAML     | The path of Tekton Pipelines YAML file               |
| INGRESS_NGINX_YAML        | The path of Ingress Nginx YAML file                  |
| CERT_MANAGER_YAML         | The path of Cert Manager YAML file                   |
| OPENFUNCTION_YAML         | The path of OpenFunction YAML file                   |

{{% alert title="Note" color="success" %}}

You can use any value supported by kubectl's `--filename` option.

{{% /alert %}}
