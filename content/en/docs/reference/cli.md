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
| `create`      | Creates a function from a file or stdin.                     |
| `apply`       | Applies a function from a file or stdin.                     |
| `demo`        | Creates a [kind cluster](https://kind.sigs.k8s.io/) to install OpenFunction and run its demo. |
| `get`         | Prints a table of the most important information about the specified function. |
| `get builder` | Prints important information about the Builder.              |
| `get serving` | Prints important information about the Serving.              |
| `delete`      | Deletes the specified function.                              |

## `ofn demo` Parameters

The following table describes parameters available for the `ofn demo` command.

| Parameter    | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| --auto-prune | For removing the demo kind cluster. To keep the demo kind cluster, run `ofn demo --auto-prune=false`, and you can delete the demo kind cluster by running `kind delete cluster --name openfunction`. |
| --verbose    | For displaying detailed information.                         |
| --region-cn  | For users who have limited access to gcr.io or github.com.   |

