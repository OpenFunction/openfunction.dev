---
title: "Function"
linkTitle: "Function"
weight: 3100
description: >	
    Learn about the concept of Function in OpenFunction.
---

This document describes the concept of Function in OpenFunction.

## Function

Function is a [Custom Resource Definition (CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) that you can define and manage. It is a description of your application, including what source code is used to build the application image and how the workload works in certain runtime environments.

Function aims to realize the lifecycle management from your source code to the final application workloads that respond to events through a single profile. It manages and coordinates [Builder](../builder/) and [Serving](../serving/) to handle details of the process.

## Reference

For more information, see [Function Specifications](../../reference/component-reference/function-spec).

For more information about OpenFunction Functions Framework, see [OpenFunction Functions Framework](https://github.com/OpenFunction/functions-framework).
