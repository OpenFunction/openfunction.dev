---
title: "EventBus Specifications"
linkTitle: "EventBus Specifications"
weight: 6131
description: >	
    EventBus Specifications.
---

This document describes the specifications of the EventBus (ClusterEventBus) CRD.

## EventBus (ClusterEventBus)

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventBus(ClusterEventBus)                                    |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[EventBusSpec](#eventbusspec)*                     | Refer to [EventBusSpec](#eventbusspec)                       |
| **status** *EventBusStatus*                                  | Status of EventBus(ClusterEventBus)                          |

### EventBusSpec

*Belong to [EventBus](#eventbus-clustereventbus)*.

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **topic** *string*                                           | The topic name of the event bus.                             |
| **natsStreaming** *[NatsStreamingSpec](../natsstreaming-spec)* | Definition of the Nats Streaming event bus (currently only supported). See [NatsStreamingSpec](../natsstreaming-spec). |

### GenericScaleOption

*Belong to scaleOption*.

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **pollingInterval** *int*                                    | The interval to check each trigger on. It defaults to `30` seconds. |
| **cooldownPeriod** *int*                                     | The period to wait after the last trigger reported active before scaling the resource back to 0. It defaults to `300` seconds. |
| **minReplicaCount** *int*                                    | Minimum number of replicas which KEDA will scale the resource down to. It defaults to `0`. |
| **maxReplicaCount** *int*                                    | This setting is passed to the HPA definition that KEDA will create for a given resource. |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | See [KEDA documentation](https://keda.sh/docs/2.4/concepts/scaling-deployments/). |
| **metadata** *map[string]string*                             | KEDA trigger's metadata.                                     |
| **authRef** *[kedav1alpha1.ScaledObjectAuthRef](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaledObjectAuthRef)* | Every parameter you define in `TriggerAuthentication` definition does not need to be included in the `metadata` of the trigger for your `ScaledObject` definition. To reference a `TriggerAuthentication` from a `ScaledObject`, add the `authRef` to the trigger. Refer to [KEDA documentation](https://keda.sh/docs/2.4/concepts/authentication/). |

