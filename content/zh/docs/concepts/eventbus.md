---
title: "EventBus（ClusterEventBus）"
linkTitle: "EventBus（ClusterEventBus）"
weight: 30
description: >
    OpenFunction 概念之事件框架资源，事件总线 EventBus（ClusterEventBus）
---

**EventBus** 负责聚合事件并持久化事件。

**EventBus** 包含对事件总线后端服务的描述（通常是一个消息服务器，如 NATS Streaming、Kafka 等），然后为 **EventSource** 和 **Trigger** 提供这些配置（让它们可以从事件总线中获取事件）。

**EventBus** 默认负责 namespace scope 的事件总线适配，同时我们也为 cluster scope 提供了一个事件总线适配器 **ClusterEventBus** ，它将在命名空间中不存在 EventBus 时生效。

### 支持的事件总线服务器

- [NATS Streaming]({{<ref "../reference/eventbus-spec/natsstreaming-spec">}})

### 参考

EventBus 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [EventBus CRD Spec]({{< ref "../reference/eventbus-spec" >}}) 了解更多信息。
