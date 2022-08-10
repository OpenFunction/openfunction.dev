---
title: "Installation"
linkTitle: "Installation"
weight: 2100
description:
---

This document describes how to install OpenFunction.

## Prerequisites

- You need to have a Kubernetes cluster.

- You need to ensure your Kubernetes version meets the requirements described in the following compatibility matrix. 

| OpenFunction Version | Kubernetes 1.17 | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20+ |
|----------------------| --------------- | --------------- |-----------------| ---------------- |
| HEAD                 | N/A             | N/A             | N/A             | √                |
| v0.7.0               | N/A             | N/A             | N/A             | √                |
| v0.6.0               | √*              | √*              | √               | √                |

## Install OpenFunction

Now you can install OpenFunction and all its dependencies with helm charts.
> The `ofn` CLI install method is deprecated.

### Requirements
- Kubernetes version: `>=v1.20.0-0`
- Helm version: `>=v3.6.3`

### Steps to install OpenFunction helm charts

1. Run the following command to add the OpenFunction chart repository first:
   ```shell
   helm repo add openfunction https://openfunction.github.io/charts/
   helm repo update
   ```

2. Then you have several options to setup OpenFunction, you can choose to:

   - Install all components:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction openfunction/openfunction -n openfunction
      ```
   
   - Install Serving only (without build):
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.ShipwrightBuild.enabled=false --set global.TektonPipelines.enabled=false openfunction/openfunction -n openfunction
      ```
   
   - Install Knative sync runtime only:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.Keda.enabled=false openfunction/openfunction -n openfunction
      ```
   
   - Install OpenFunction async runtime only:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.Contour.enabled=false  --set global.KnativeServing.enabled=false openfunction/openfunction -n openfunction
      ```

   {{% alert title="Note" color="success" %}}

   For more information about how to install OpenFunction with Helm, see [Install OpenFunction with Helm](https://github.com/OpenFunction/charts#install-the-chart).

   {{% /alert %}}

3. Run the following command to verify OpenFunction is up and running:
   ```shell
   kubectl get po -n openfunction
   ```

## Uninstall OpenFunction
### Helm
If you installed OpenFunction with Helm, run the following command to uninstall OpenFunction and its dependencies.
```shell
helm uninstall openfunction -n openfunction
```
{{% alert title="Note" color="success" %}}

For more information about how to uninstall OpenFunction with Helm, see [Uninstall OpenFunction with Helm](https://github.com/OpenFunction/charts#uninstall-the-chart).

{{% /alert %}}