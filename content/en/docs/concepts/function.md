---
title: "Function"
linkTitle: "Function"
weight: 10
description: >
    Concept of OpenFunction, a resource of the serverless framework, Function
---

**Function** is a resource that is directly defined and controlled by the user. It is a description of the user's application, i.e. what raw materials (source code) are used to make the product (application image) and how it will ultimately work (workload, runtime).

In OpenFunction, the **Function** resource controls the coordination of **[Builder]({{<ref "builder">}})** and **[Serving]({{<ref "serving">}})** in an orderly manner according to the configuration, thus implementing the lifecycle management of user functions.

### Reference

Function is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [Function CRD Spec]({{< ref "../reference/function-spec" >}}) for more information.

