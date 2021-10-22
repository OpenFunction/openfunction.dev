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

### GenericScaleOption

*Belong to scaleOption*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **pollingInterval** *int*                                    | This is the interval to check each trigger on. Default is `30` seconds |
| **cooldownPeriod** *int*                                     | The period to wait after the last trigger reported active before scaling the resource back to 0. Default is `300` seconds |
| **minReplicaCount** *int*                                    | Minimum number of replicas KEDA will scale the resource down to. Default is `0` |
| **maxReplicaCount** *int*                                    | This setting is passed to the HPA definition that KEDA will create for a given resource |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | See [KEDA docs](https://keda.sh/docs/2.4/concepts/scaling-deployments/) |
| **metadata** *map[string]string*                             | KEDA trigger's metadata                                      |
| **authRef** *[kedav1alpha1.ScaledObjectAuthRef](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaledObjectAuthRef)* | Every parameter you define in `TriggerAuthentication` definition does not need to be included in the `metadata` of the trigger for your `ScaledObject` definition. To reference a `TriggerAuthentication` from a `ScaledObject` you add the `authRef` to the trigger, refer to [KEDA docs](https://keda.sh/docs/2.4/concepts/authentication/) |
