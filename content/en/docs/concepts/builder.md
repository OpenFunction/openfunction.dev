---
title: "Builder"
linkTitle: "Builder"
weight: 3200
description: >	
    Learn about the concept of Builder in OpenFunction.
---

This document describes the concept of Builder in OpenFunction.

## Builder

Builder is a [Custom Resource Definition (CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) that defines the build work in OpenFunction for generating application images from source code.

Currently, Builder uses [Shipwright](https://github.com/shipwright-io/build) and [Cloud Native Buildpacks](https://buildpacks.io/) to build your function source code into application images. It controls the process of building application images through Shipwright, including acquiring code, generating image artifacts, and publishing images through Cloud Native Buildpacks.

The following languages are supported:

| Runtime | Serving Support        | Eventing Type Support | Version                                                      |
| ------- | ---------------------- | --------------------- | ------------------------------------------------------------ |
| Go      | OpenFuncAsync, Knative | HTTP, CloudEvent      | [1.15](https://github.com/OpenFunction/builder/blob/main/builders/go115), [1.16](https://github.com/OpenFunction/builder/blob/main/builders/go116) |
| Node.js | Knative                | HTTP                  | [10](https://github.com/OpenFunction/builder/blob/main/builders/node10), [12](https://github.com/OpenFunction/builder/blob/main/builders/node12), [14](https://github.com/OpenFunction/builder/blob/main/builders/node14), [16](https://github.com/OpenFunction/builder/blob/main/builders/node16) |
| Python  | Knative                | HTTP                  | [3.7](https://github.com/OpenFunction/builder/blob/main/builders/py37), [3.8](https://github.com/OpenFunction/builder/blob/main/builders/py38), [3.9](https://github.com/OpenFunction/builder/blob/main/builders/py39) |
| Java    | Knative                | HTTP                  | [11](https://github.com/OpenFunction/builder/blob/main/builders/java11) |

