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

### GenericScaleOption

*从属 scaleOption*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **pollingInterval** *int*                                    | 检查每个伸缩触发器的时间间隔。默认是`30`秒                   |
| **cooldownPeriod** *int*                                     | 在将资源缩减到 0 之前，最后一个伸缩触发器报告激活后的等待时间。 默认为 `300` 秒 |
| **minReplicaCount** *int*                                    | KEDA 将收缩资源的最小副本数。默认值是 `0`                    |
| **maxReplicaCount** *int*                                    | 这个设置被传递给KEDA为特定资源创建的HPA定义，KEDA 扩展资源的最大副本数。 |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | 请查看 [KEDA 文档](https://keda.sh/docs/2.4/concepts/scaling-deployments/) |
| **metadata** *map[string]string*                             | KEDA 伸缩触发器的元数据                                      |
| **authRef** *[kedav1alpha1.ScaledObjectAuthRef](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaledObjectAuthRef)* | 你在 `TriggerAuthentication` 配置中定义的每个参数都不需要包含在你的 `ScaledObject` 定义的触发器的 `metadata` 中。要从 `ScaledObject` 中引用 `TriggerAuthentication`，你需要在触发器中添加 `authRef`，请参考 [KEDA 文档](https://keda.sh/docs/2.4/concepts/authentication/) |
