---
title: "EventSource"
linkTitle: "EventSource"
weight: 25
description: >
    OpenFunction 概念之事件框架资源，事件源 EventSource
---

**EventSource** 代表一个事件的生产者，如 Kafka 服务器、对象存储服务，也可以是一个应用。它包含这些事件生产者的定义，以便于从这些生产者处获取事件。它同时定义了这些事件将去往何处。

### 支持的事件源

- [Kafka]({{<ref "../reference/eventsource-spec/kafka-spec">}})
- [Cron (scheduler)]({{<ref "../reference/eventsource-spec/cron-spec">}})
- [Redis]({{<ref "../reference/eventsource-spec/redis-spec">}})

### 参考

EventSource 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [EventSource CRD Spec]({{< ref "../reference/eventsource-spec" >}}) 了解更多信息。
