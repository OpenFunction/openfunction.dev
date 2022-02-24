---
title: "OpenFunction CLI"
linkTitle: "OpenFunction CLI"
weight: 4100
description: >	
    Learn how to use the CLI of OpenFunction.
---

This document describes how to use the CLI of OpenFunction.

## Overview

ofn is the command-line interface for OpenFunction. It helps you install and manage OpenFuntion.

The following table describes the main commands supported by ofn.

| Command       | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| `init`        | Provides management for the framework of OpenFunction.       |
| `install`     | Installs OpenFunction and its dependencies.                  |
| `uninstall`   | Uninstalls OpenFunction and its dependencies.                |
| `create`      | Creates a function from a file or stdin.                     |
| `apply`       | Applies a function from a file or stdin.                     |
| `demo`        | Creates a [kind cluster](https://kind.sigs.k8s.io/) to install OpenFunction and run its demo. |
| `get`         | Prints a table of the most important information about the specified function. |
| `get builder` | Prints important information about the Builder.              |
| `get serving` | Prints important information about the Serving.              |
| `delete`      | Deletes the specified function.                              |

## Use ofn to Install and Uninstall OpenFunction

1. Run the following command to download ofn.

   ```shell
   wget -c  https://github.com/OpenFunction/cli/releases/latest/download/ofn_linux_amd64.tar.gz -O - | tar -xz
   ```

2. Run un the following commands to make `ofn` executable and move it to `/usr/local/bin/`.

   ```shell
   chmod +x ofn && mv ofn /usr/local/bin/
   ```

   {{% alert title="Note" color="success" %}}

   If you download ofn from [the release page](https://github.com/OpenFunction/cli/releases), make sure you put ofn in the correct directory.

   {{% /alert %}}

3. Run the following command to install OpenFunction.

   ```shell
   ofn install --all
   ```

3. Run the following command to uninstall OpenFunction and its dependencies.

   ```shell
   ofn uninstall --all
   ```

   {{% alert title="Note" color="success" %}}

   You can also remove the `--all` parameter to just uninstall OpenFunction.

   {{% /alert %}}

## `ofn install` Parameters

The following table describes parameters available for the `ofn install` command.

| Parameter                         | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| --all                             | For installing all dependencies.                             |
| --async                           | For installing OpenFunction Async runtime (Dapr & KEDA).     |
| --cert-manager                    | For installing Cert Manager.                                 |
| --dapr                            | For installing Dapr.                                         |
| --dry-run                         | For previewing the components and their versions to be installed. |
| --ingress                         | For installing Ingress Nginx.                                |
| --keda                            | For installing KEDA.                                         |
| --knative                         | For installing Knative Serving (with Kourier as the default gateway). |
| --region-cn                       | For users who have limited access to gcr.io or github.com. If you add this parameter in the `ofn install` command, you have to add it again in the `ofn uninstall` command. |
| --shipwright                      | For installing Shipwright.                                   |
| --sync                            | For installing OpenFunction Sync runtime (to be supported).  |
| --upgrade                         | For upgrading components to target versions during installation. |
| --yes                             | For automatically passing yes to prompts. (`-y` for short)   |
| --verbose                         | For displaying detailed information.                         |
| --version &lt; version number&gt; | For specifying any stable version or the latest version of OpenFunction for installation. (default "v0.4.0") If not specified, the latest stable version will be installed. |
| --timeout &lt; duration &gt;      | For specifying the timeout duration, which defaults to 5 minutes. |

## `ofn uninstall` Parameters

The following table describes parameters available for the `ofn uninstall` command.

| Parameter                         | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| --all                             | For uninstalling all dependencies.                           |
| --async                           | For uninstalling OpenFunction Async runtime (Dapr & KEDA).   |
| --cert-manager                    | For uninstalling Cert Manager.                               |
| --dapr                            | For uninstalling Dapr.                                       |
| --dry-run                         | For previewing the components and their versions to be uninstalled. |
| --ingress                         | For uninstalling Ingress Nginx.                              |
| --keda                            | For uninstalling KEDA.                                       |
| --knative                         | For uninstalling Knative Serving (with Kourier as the default gateway). |
| --region-cn                       | For users who have limited access to gcr.io or github.com. If you add this parameter in the `ofn install` command, you have to add it again in the `ofn uninstall` command. |
| --shipwright                      | For uninstalling Shipwright.                                 |
| --sync                            | For uninstalling OpenFunction Sync runtime (to be supported). |
| --verbose                         | For displaying detailed information.                         |
| --yes                             | For automatically passing yes to prompts. (`-y` for short)   |
| --version &lt; version number&gt; | For specifying any stable version or the latest version of OpenFunction to be uninstalled. If not specified, the currently installed version will be uninstalled. |
| --wait                            | For awaiting the results of the uninstallation until namespaces are cleaned up. |
| --timeout &lt; duration &gt;      | For specifying the timeout duration, which defaults to 5 minutes. |

### Inventory

During installation, the OpenFunction CLI keeps the installed component details in `$home/.ofn/<cluster name>-inventory.yaml`. Therefore, during uninstallation, the OpenFunction CLI removes the installed components based on the contents of `$home/.ofn/<cluster name>-inventory.yaml`.

In addition, the OpenFunction CLI supports obtaining component versions and paths to the component YAML files from environment variables.

## `ofn demo` Parameters

The following table describes parameters available for the `ofn demo` command.

| Parameter    | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| --auto-prune | For removing the demo kind cluster. To keep the demo kind cluster, run `ofn demo --auto-prune=false`, and you can delete the demo kind cluster by running `kind delete cluster --name openfunction`. |
| --verbose    | For displaying detailed information.                         |
| --region-cn  | For users who have limited access to gcr.io or github.com.   |

