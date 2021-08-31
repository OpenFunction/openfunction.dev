---
title: "Serving"
linkTitle: "Serving"
weight: 20
description: >
    Concept of OpenFunction, a resource of the serverless framework, Serving
---

The goal of **Serving** is to run applications in a highly elastic way (dynamic scaling: 0 <-> N).

Currently, OpenFunction Serving supports two kinds of workload runtimes: [Knative](#knative) and [OpenFuncAsync](#openfuncasync). **Serving** will only work when one of the them is set.

### Knative

[Knative Serving](https://github.com/knative/serving) is built on overlay of Kubernetes to support deploying and running Serverless applications. Knative Serving is easy to get started with and can be extended to support complex application scenarios.

### OpenFuncAsync

**OpenFuncAsync** is an event-driven workload runtime. It is implemented by [KEDA](https://keda.sh/) and [Dapr](https://dapr.io/). Functions of the OpenFuncAsync runtime can be triggered by various events, messages, such as MQ, cronjob, DB events, etc. In a Kubernetes cluster, functions of OpenFuncAsync runtime can be run as stateless workloads or tasks.

### Reference

Serving is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [Serving CRD Spec]({{< ref "../reference/function-spec#servingimpl" >}}) for more information.
