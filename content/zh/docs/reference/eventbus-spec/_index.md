---
title: "EventBus 参数说明"
linkTitle: "EventBus 参数说明"
weight: 15
description: >
    EventBus 及 ClusterEventBus CRD 参数规范说明
---

## EventBus(ClusterEventBus)

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventBus(ClusterEventBus)                                    |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *（可选）* 参考 [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) 文档 |
| **spec** *[EventBusSpec](#eventsourcespec)*                  | 事件总线的规格，参考 [EventBusSpec](#eventsourcespec)        |
| **status** *EventBusStatus*                                  | 事件总线的状态                                               |

### EventBusSpec

*从属 [EventBus](#eventbus)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **topic** *string*                                           | 事件总线的 topic 名称                                        |
| **natsStreaming** *[NatsStreamingSpec]({{<ref "natsstreaming-spec.md#natsstreamingspec" >}})* | （当前仅支持）nats streaming 事件总线的定义，参考 [NatsStreamingSpec]({{<ref "natsstreaming-spec.md#natsstreamingspec" >}}) |

