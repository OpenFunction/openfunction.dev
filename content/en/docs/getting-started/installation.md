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

| OpenFunction Version | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 | Kubernetes 1.23 | Kubernetes 1.24 | Kubernetes 1.25 |
|----------------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| HEAD                 | N/A             | √               | √               | √               |  √              | √               |
| v1.1.x               | N/A             | √               | √               | √               |  √              | √              |
| v1.0.x               | N/A             | √               | √               | √               |  √              | √              |
| v0.8.x               | √               | √               | √               | √               |  √              | √*              |

{{% alert title="Note" color="success" %}}

OpenFunction supports Kubernetes 1.25 starting from v0.8.1

{{% /alert %}}


## Install OpenFunction

Now you can install OpenFunction and all its dependencies with helm charts.
> The `ofn` CLI install method is deprecated.

If you want to install OpenFunction in an offline environment, please refer to [Install OpenFunction in an offline environment](https://openfunction.dev/docs/reference/faq/#q-how-to-install-openfunction-in-an-offline-environment)

### Requirements
- Kubernetes version: `>=v1.21.0-0`
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
     
   - Install all components and [Revision Controller](https://openfunction.dev/docs/concepts/cicd/):
      ```shell
      kubectl create namespace openfunction
      helm install openfunction openfunction/openfunction -n openfunction --set revisionController.enable=true
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

## Upgrade OpenFunction

```shell
helm upgrade [RELEASE_NAME] openfunction/openfunction -n openfunction
```

With Helm v3, CRDs created by this chart are not updated by default and should be manually updated.
See also the [Helm Documentation on CRDs](https://helm.sh/docs/chart_best_practices/custom_resource_definitions).

_Refer to [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) for command documentation._

### Upgrading an existing Release to a new version

### From OpenFunction v0.6.0 to OpenFunction v0.7.x

There is a breaking change when upgrading from v0.6.0 to 0.7.x which requires additional manual operations.
#### Uninstall the Chart

First, you'll need to uninstall the old `openfunction` release:
```shell
helm uninstall openfunction -n openfunction
```

Confirm that the component namespaces have been deleted, it will take a while:
```shell
kubectl get ns -o=jsonpath='{range .items[?(@.metadata.annotations.meta\.helm\.sh/release-name=="openfunction")]}{.metadata.name}: {.status.phase}{"\n"}{end}'
```

> If the knative-serving namespace is in the terminating state for a long time, try running the following command and remove finalizers:
```shell
kubectl edit ingresses.networking.internal.knative.dev -n knative-serving
```

#### Upgrade OpenFunction CRDs
Then you'll need to upgrade the new OpenFunction CRDs

```shell
kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/openfunction.yaml
```

#### Upgrade dependent components CRDs
You also need to upgrade the dependent components' CRDs
> You only need to deal with the components included in the existing Release.
- knative-serving CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/knative-serving.yaml
    ```
- shipwright-build CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/shipwright-build.yaml
    ```
- tekton-pipelines CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/tekton-pipelines.yaml
    ```

#### Install new chart
```shell
helm repo update
helm install openfunction openfunction/openfunction -n openfunction
```

{{% alert title="Note" color="success" %}}

For more information about how to upgrade OpenFunction with Helm, see [Upgrade OpenFunction with Helm](https://github.com/OpenFunction/charts#upgrading-chart).

{{% /alert %}}