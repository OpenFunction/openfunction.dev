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
| -------------------- | --------------- | --------------- | --------------- | ---------------- |
| v0.6.0               | √               | √               | √               | √                |
| v0.5.0               | √               | √               | √               | √                |
| v0.4.0               | √               | √               | √               | √                |
| HEAD                 | √               | √               | √               | √                |

{{% alert title="Note" color="success" %}}

OpenFunction has added the [Domain](../../concepts/domain) feature since v0.5.0. To use this feature, you have to install OpenFunction in Kubernetes 1.19 or later. For more information about OpenFunction components compatibility with Kubernetes, refer to [Component Compatibility Matrix](../../best-practices/customize-components#component-compatibility-matrix).

{{% /alert %}}

## Install OpenFunction

### Option 1: Helm
This option is the **recommended** installation method.

#### Requirements
- Kubernetes version: `>=v1.20.0-0`
- Helm version: `>=v3.6.3`

#### install the chart
1. Run the following command to add the OpenFunction chart repository.
   ```shell
   helm repo add openfunction https://openfunction.github.io/charts/
   helm repo update
   ```

2. Run the following command to install the OpenFunction chart.
   ```shell
   kubectl create namespace openfunction
   helm install openfunction openfunction/openfunction -n openfunction
   ```
   {{% alert title="Note" color="success" %}}

   For more information about how to install OpenFunction with Helm, see [Install OpenFunction with Helm](https://github.com/OpenFunction/charts/blob/main/README.md#install-the-chart).

   {{% /alert %}}

3. Run the following command to verify OpenFunction is ready.
   ```shell
   kubectl get pods -namespace openfunction
   ```

### Option 2: CLI
1. Run the following command to download `ofn`, the CLI of OpenFunction.

   ```shell
   wget -c  https://github.com/OpenFunction/cli/releases/latest/download/ofn_linux_amd64.tar.gz -O - | tar -xz
   ```

2. Run the following commands to make `ofn` executable and move it to `/usr/local/bin/`.

   ```shell
   chmod +x ofn && mv ofn /usr/local/bin/
   ```

3. Run the following command to install OpenFunction.

   ```shell
   ofn install --all
   ```

   {{% alert title="Note" color="success" %}}

   For more information about how to use the `ofn install` command, see [`ofn install` Parameters](../../user-guide/cli#ofn-install-parameters).

   {{% /alert %}}

## Uninstall OpenFunction
### Helm
If you installed OpenFunction with Helm, run the following command to uninstall OpenFunction and its dependencies.
```shell
helm uninstall openfunction -n openfunction
```
{{% alert title="Note" color="success" %}}

For more information about how to uninstall OpenFunction with Helm, see [Uninstall OpenFunction with Helm](https://github.com/OpenFunction/charts/blob/main/README.md#uninstall-the-chart).

{{% /alert %}}

### CLI
If you installed OpenFunction with CLI, run the following command to uninstall OpenFunction and its dependencies.
```shell
ofn uninstall --all
```

{{% alert title="Note" color="success" %}}

For more information about how to use the `ofn uninstall` command, see [`ofn uninstall` Parameters](../../user-guide/cli#ofn-uninstall-parameters).

{{% /alert %}}

