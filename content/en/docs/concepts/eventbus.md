---
title: "EventBus(ClusterEventBus)"
linkTitle: "EventBus(ClusterEventBus)"
weight: 30
description: >
    Concept of OpenFunction, a resource of the events framework, EventBus(ClusterEventBus)
---

**EventBus** is responsible for aggregating and persisting events.

**EventBus** contains a description of the event bus backend service (usually a message server such as NATS Streaming, Kafka, etc.) and then provides these configurations for **EventSource** and **Trigger** (allowing them to fetch events from the event bus).

**EventBus** is responsible for event bus adaptation in the namespace scope by default, and we also provide an event bus adapter **ClusterEventBus** for the cluster scope which will take effect when no EventBus exists in the namespace.

### Available Event Bus Servers

- [NATS Streaming]({{<ref "../reference/eventbus-spec/natsstreaming-spec">}})

### Reference

EventBus is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [EventBus CRD Spec]({{< ref "../reference/eventbus-spec" >}}) for more information.
