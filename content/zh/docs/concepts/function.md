---
title: "Function"
linkTitle: "Function"
weight: 10
description: >
    OpenFunction 概念之核心框架资源，函数实例 Function
---

**Function** 是直接由使用者定义、控制的资源，它是使用者对其业务应用的一段描述，即用何种原料（源代码）加工成何种制品（应用镜像），最终又将以何种方式运作（工作负载、运行时）。

在 OpenFunction 中，**Function** 资源会根据配置有序控制 **[Builder]({{<ref "builder">}})** 和 **[Serving]({{<ref "serving">}})** 资源的协调过程，进而实现使用者函数的生命周期管理。

### 参考

Function 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [Function CRD Spec]({{< ref "../reference/function-spec" >}}) 了解更多信息。

