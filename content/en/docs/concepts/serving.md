---
title: "Serving"
linkTitle: "Serving"
weight: 3300
description: >	
    Learn about the concept of Serving in OpenFunction.
---

This document describes the concept of Serving in OpenFunction.

## Serving

Serving a [Custom Resource Definition (CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) that aims to run functions in a highly elastic way through dynamic scaling. Currently, Serving supports two kinds of workload runtimes, [Knative Serving](https://github.com/knative/serving) and OpenFuncAsync.

### Knative Serving

Knative Serving builds on Kubernetes to support deploying and serving Serverless applications. It is easy to get started with and can be extended to support complex application scenarios.

### OpenFuncAsync

OpenFuncAsync is an event-driven Serving runtime. It is implemented based on [KEDA](https://keda.sh/) and [Dapr](https://dapr.io/). Functions of the OpenFuncAsync runtime can be triggered by various event types, such as MQ, cronjob, and DB events. In a Kubernetes cluster, OpenFuncAsync functions run in the form of deployments or jobs.



