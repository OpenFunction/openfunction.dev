---
title: "EventSource Spec"
linkTitle: "EventSource Spec"
weight: 10
description: >
    EventSource CRD Specification
---

## EventSource

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventSource                                                  |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[EventSourceSpec](#eventsourcespec)*               | Refer to [EventSourceSpec](#eventsourcespec)                 |
| **status** *EventSourceStatus*                               | Status of EventSource                                        |

### EventSourceSpec

*Belong to [EventSource](#eventsource)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **eventBus** *string*                                        | *(Optional)* Name of the EventBus resource associated with the event source |
| **redis** *map\[string][RedisSpec]({{<ref "redis-spec.md#redisspec" >}})* | *(Optional)* The definition of a Redis event source, key represents the event name, refer to [RedisSpec]({{<ref "redis-spec.md#redisspec" >}}) |
| **kafka** *map\[string][KafkaSpec]({{<ref "kafka-spec.md#kafkaspec" >}})* | *(Optional)* The definition of a Kafka event source, key represents the event name, refer to [KafkaSpec]({{<ref "kafka-spec.md#kafkaspec" >}}) |
| **cron** *map\[string][CronSpec]({{<ref "cron-spec.md#cronspec" >}})* | *(Optional)* The definition of a Cron event source, key represents the event name, refer to [CronSpec]({{<ref "cron-spec.md#cronspec" >}}) |
| **sink** *[SinkSpec](#sinkspec)*                             | *(Optional)* Definition of the Sink (addressable access resource, i.e. synchronization request) associated with the event source, cf. [SinkSpec](#sinkspec) |

### SinkSpec

*Belong to [EventSourceSpec](#eventsourcespec)*

| Field                             | Description                      |
| --------------------------------- | -------------------------------- |
| **ref** *[Reference](#reference)* | Refer to [Reference](#reference) |

### Reference

*Belong to [SinkSpec](#sinkspec)*

> The resources cited are generally [Knative Service](https://knative.dev/docs/reference/api/serving-api/#serving.knative.dev/v1.Service) 

| Field                   | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **kind** *string*       | The type of the referenced resource. Default is `Service`    |
| **namespace** *string*  | The namespace of the referenced resource, by default the same as the namespace of the Trigger |
| **name** *string*       | Name of the referenced resource, e.g. `function-ksvc`        |
| **apiVersion** *string* | The apiVersion of the referenced resource. Default is `serving.knative.dev/v1` |