---
title: "EventSource"
linkTitle: "EventSource"
weight: 25
description: >
    Concept of OpenFunction, a resource of the events framework, EventSource
---

**EventSource** represents a producer of events, such as a Kafka server, an object storage service, or it can be an application. It contains definitions of these event producers to facilitate the fetching of events from these producers. It also defines the destination of these events.

### Available event sources

- [Kafka]({{<ref "../reference/eventsource-spec/kafka-spec">}})
- [Cron (scheduler)]({{<ref "../reference/eventsource-spec/cron-spec">}})
- [Redis]({{<ref "../reference/eventsource-spec/redis-spec">}})

### Reference

EventSource is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [EventSource CRD Spec]({{< ref "../reference/eventsource-spec" >}}) for more information.

