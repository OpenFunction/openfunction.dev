---
title: "EventSource Specifications"
linkTitle: "EventSource Specifications"
weight: 6121
description: >	
    EventSource Specifications.
---

This document describes the specifications of the EventSource CRD.

## EventSource

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventSource                                                  |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[EventSourceSpec](#eventsourcespec)*               | Refer to [EventSourceSpec](#eventsourcespec)                 |
| **status** *EventSourceStatus*                               | Status of EventSource                                        |

### EventSourceSpec

*Belong to [EventSource](#eventsource)*.

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **eventBus** *string*                                        | *(Optional)* Name of the EventBus resource associated with the event source. |
| **redis** *map\[string][RedisSpec]({{<ref "redis-spec.md#redisspec" >}})* | *(Optional)* The definition of a Redis event source, with key being the event name, refer to [RedisSpec](../redis-spec). |
| **kafka** *map\[string][KafkaSpec]({{<ref "kafka-spec.md#kafkaspec" >}})* | *(Optional)* The definition of a Kafka event source, with key being the event name, refer to [KafkaSpec](../kafka-spec). |
| **cron** *map\[string][CronSpec]({{<ref "cron-spec.md#cronspec" >}})* | *(Optional)* The definition of a Cron event source, with key being the event name, refer to [CronSpec](../cron-spec). |
| **sink** *[SinkSpec](#sinkspec)*                             | *(Optional)* Definition of the Sink (addressable access resource, i.e. synchronization request) associated with the event source, cf. [SinkSpec](#sinkspec). |

### SinkSpec

*Belong to [EventSourceSpec](#eventsourcespec)*.

| Field                             | Description                       |
| --------------------------------- | --------------------------------- |
| **ref** *[Reference](#reference)* | Refer to [Reference](#reference). |

### Reference

*Belong to [SinkSpec](#sinkspec)*.

{{% alert title="Note" color="success" %}}

The resources cited are generally [Knative Service](https://knative.dev/docs/reference/api/serving-api/#serving.knative.dev/v1.Service).

{{% /alert %}}

| Field                   | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **kind** *string*       | The type of the referenced resource. It defaults to `Service`. |
| **namespace** *string*  | The namespace of the referenced resource, by default the same as the namespace of the Trigger. |
| **name** *string*       | Name of the referenced resource, for example, `function-ksvc`. |
| **apiVersion** *string* | The apiVersion of the referenced resource. It defaults to `serving.knative.dev/v1`. |

### GenericScaleOption

*Belong to scaleOption*.

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **pollingInterval** *int*                                    | This is the interval to check each trigger on. It defaults to `30` seconds. |
| **cooldownPeriod** *int*                                     | The period to wait after the last trigger reported active before scaling the resource back to 0. It defaults to `300` seconds. |
| **minReplicaCount** *int*                                    | Minimum number of replicas KEDA will scale the resource down to. It defaults to `0`. |
| **maxReplicaCount** *int*                                    | This setting is passed to the HPA definition that KEDA will create for a given resource. |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | See [KEDA documentation](https://keda.sh/docs/2.4/concepts/scaling-deployments/). |
| **metadata** *map[string]string*                             | KEDA trigger's metadata                                      |
| **authRef** *[kedav1alpha1.ScaledObjectAuthRef](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaledObjectAuthRef)* | Every parameter you define in `TriggerAuthentication` definition does not need to be included in the `metadata` of the trigger for your `ScaledObject` definition. To reference a `TriggerAuthentication` from a `ScaledObject` you add the `authenticationRef` to the trigger, refer to [KEDA documentation](https://keda.sh/docs/2.4/concepts/authentication/). |

