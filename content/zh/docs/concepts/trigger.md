---
title: "Trigger"
linkTitle: "Trigger"
weight: 35
description: >
    OpenFunction 概念之事件框架资源，事件触发器 Trigger
---

**Trigger** 是对事件目的的抽象，即在收到一个消息时需要做什么。

**Trigger** 包含了使用者对事件目的的描述，它指导 Trigger 应该从哪些事件源获取事件，随后根据给定的条件决定是否触发目标函数。

### 参考

Trigger 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [Trigger CRD Spec]({{< ref "../reference/trigger-spec" >}}) 了解更多信息。  

