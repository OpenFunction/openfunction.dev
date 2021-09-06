---
title: "Serving"
linkTitle: "Serving"
weight: 20
description: >
    OpenFunction 概念之核心框架资源，函数负载器 Serving
---

**Serving** 的目标是以高度弹性的方式（动态伸缩：0 <-> N）为使用者运行应用负载。

当前，OpenFunction Serving 支持两种负载运行时：[Knative](#knative) 和 [OpenFuncAsync](#openfuncasync)。设置其中一种负载运行时之后，Serving 才能正常工作。

### Knative

[Knative Serving](https://github.com/knative/serving) 建立在 Kubernetes 之上，支持部署和运行 Serverless 应用。Knative Serving 很容易上手，并可延伸支持复杂的应用场景。

### OpenFuncAsync

OpenFuncAsync 是一个事件驱动的负载运行时。它基于 [KEDA](https://keda.sh/) 与 [Dapr](https://dapr.io/) 实现。OpenFuncAsync 负载的函数可以由各类事件、消息触发，如 MQ、cronjob、DB 事件等。在 Kubernetes 集群中，OpenFuncAsync 负载的函数可以通过无状态工作负载或任务的形式运行。

### 参考

Serving 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [Serving CRD Spec]({{< ref "../reference/function-spec#servingimpl" >}}) 了解更多信息。
