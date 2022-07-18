---
title: "Trigger Specifications"
linkTitle: "Trigger Specifications"
weight: 6140
description: >	
    Learn about Trigger Specifications.
---

This document describes the specifications of the Trigger CRD.

## Trigger

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | Trigger                                                      |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[TriggerSpec](#triggerspec)*                       | Refer to [TriggerSpec](#triggerspec)                         |
| **status** *TriggerStatus*                                   | Status of Trigger                                            |

### TriggerSpec

*Belong to [Trigger](#trigger)*.

| Field                                          | Description                                                  |
| ---------------------------------------------- | ------------------------------------------------------------ |
| **eventBus** *string*                          | *(Optional)* Name of the EventBus resource associated with the Trigger. |
| **inputs** *map\[string][Input](#input)*       | *(Optional)* The input of trigger, with key being the input name. Refer to [Input](#input). |
| **subscribers** *\[][Subscriber](#subscriber)* | *(Optional)* The subscriber of the trigger. Refer to [Subscriber](#subscriber). |

### Input

*Belong to [TriggerSpec](#triggerspec)*.

| Field                    | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **namespace** *string*   | *(Optional)* The namespace name of the EventSource, which by default matches the namespace of the Trigger, for example, `default`. |
| **eventSource** *string* | EventSource name, for example, `kafka-eventsource`.          |
| **event** *string*       | Event name, for example, `eventA`.                           |

### Subscriber

*Belong to [TriggerSpec](#triggerspec)*.

| Field                                      | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| **condition** *string*                     | Trigger conditions for triggers, refer to [cel-spec](https://github.com/google/cel-spec/blob/master/doc/langdef.md) for more writing specifications, for example, `eventA && eventB` or `eventA ||eventB`. |
| **sink** *[SinkSpec](#sinkspec)*           | *(Optional)* Triggered Sink (addressable access resource, for example, synchronization request) definition, refer to [SinkSpec](#sinkspec). |
| **deadLetterSink** *[SinkSpec](#sinkspec)* | *(Optional)* Triggered dead letter Sink (addressable access resource, for example, synchronization request) definition, refer to [SinkSpec](#sinkspec). |
| **topic** *string*                         | *(Optional)* Used to send post-trigger messages to the specified topic of the event bus, for example, `topicTriggered`. |
| **deadLetterTopic** *string*               | *(Optional)* Used to send post-trigger messages to the specified dead letter topic of the event bus, for example, `topicDL`. |

### SinkSpec

*Belong to [EventSourceSpec](../eventsource-spec/eventsource-spec/)*.

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

