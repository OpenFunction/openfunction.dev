---
title: "Trigger Spec"
linkTitle: "Trigger Spec"
weight: 20
description: >
    Trigger CRD Specification
---

## Trigger

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | Trigger                                                      |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[TriggerSpec](#triggerspec)*                       | Refer to [TriggerSpec](#triggerspec)                         |
| **status** *TriggerStatus*                                   | Refer to TriggerStatus                                       |

### TriggerSpec

*Belong to [Trigger](#trigger)*

| Field                                          | Description                                                  |
| ---------------------------------------------- | ------------------------------------------------------------ |
| **eventBus** *string*                          | *(Optional)* Name of the EventBus resource associated with the Trigger |
| **inputs** *map\[string][Input](#input)*       | *(Optional)* The input of trigger, key represents the input name, refer to [Input](#input) |
| **subscribers** *\[][Subscriber](#subscriber)* | *(Optional)* The subscriber of the trigger, refer to [Subscriber](#subscriber) |

### Input

*Belong to [TriggerSpec](#triggerspec)*

| Field                    | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **namespace** *string*   | *(Optional)* The namespace name of the EventSource, which by default matches the namespace of the Trigger. E.g. `default` |
| **eventSource** *string* | EventSource name, e.g. `kafka-eventsource`                   |
| **event** *string*       | Event name, e.g. `eventA`                                    |

### Subscriber

*Belong to [TriggerSpec](#triggerspec)*

| Field                                      | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| **condition** *string*                     | Trigger conditions for triggers, refer to [cel-spec](https://github.com/google/cel-spec/blob/master/doc/langdef.md) for more writing specifications. E.g. `eventA && eventB`, `eventA ||eventB` |
| **sink** *[SinkSpec](#sinkspec)*           | *(Optional)* Triggered Sink (addressable access resource, i.e. synchronization request) definition, refer to [SinkSpec](#sinkspec) |
| **deadLetterSink** *[SinkSpec](#sinkspec)* | *(Optional)* Triggered dead letter Sink (addressable access resource, i.e., synchronization request) definition, refer to [SinkSpec](#sinkspec) |
| **topic** *string*                         | *(Optional)* Used to send post-trigger messages to the specified topic of the event bus, e.g. `topicTriggered` |
| **deadLetterTopic** *string*               | *(Optional)* Used to send post-trigger messages to the specified dead letter topic of the event bus, e.g. `topicDL` |

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
