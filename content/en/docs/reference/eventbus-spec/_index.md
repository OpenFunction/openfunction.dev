---
title: "EventBus Spec"
linkTitle: "EventBus Spec"
weight: 15
description: >
    EventBus & ClusterEventBus CRD Specification
---

## EventBus(ClusterEventBus)

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventBus(ClusterEventBus)                                    |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[EventBusSpec](#eventbusspec)*                     | Refer to [EventBusSpec](#eventbusspec)                       |
| **status** *EventBusStatus*                                  | Status of EventBus(ClusterEventBus)                          |

### EventBusSpec

*Belong to [EventBus](#eventbusclustereventbus)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **topic** *string*                                           | The topic name of the event bus                              |
| **natsStreaming** *[NatsStreamingSpec]({{<ref "natsstreaming-spec.md#natsstreamingspec" >}})* | Definition of the Nats Streaming event bus (currently only supported), see [NatsStreamingSpec]({{<ref "natsstreaming-spec.md#natsstreamingspec" >}}) |

