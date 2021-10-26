---
title: "EventSource 参数说明"
linkTitle: "EventSource 参数说明"
weight: 10
description: >
    EventSource CRD 参数规范说明
---

## EventSource

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | EventSource                                                  |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *（可选）* 参考 [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) 文档 |
| **spec** *[EventSourceSpec](#eventsourcespec)*               | 事件源的规格，参考 [EventSourceSpec](#eventsourcespec)       |
| **status** *EventSourceStatus*                               | 事件源的状态                                                 |

### EventSourceSpec

*从属 [EventSource](#eventsource)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **eventBus** *string*                                        | *（可选）* 事件源关联的 EventBus 资源名称                    |
| **redis** *map\[string][RedisSpec]({{<ref "redis-spec.md#redisspec" >}})* | *（可选）* redis 事件源的定义，参考 [RedisSpec]({{<ref "redis-spec.md#redisspec" >}}) |
| **kafka** *map\[string][KafkaSpec]({{<ref "kafka-spec.md#kafkaspec" >}})* | *（可选）* kafka 事件源的定义，参考 [KafkaSpec]({{<ref "kafka-spec.md#kafkaspec" >}}) |
| **cron** *map\[string][CronSpec]({{<ref "cron-spec.md#cronspec" >}})* | *（可选）* cron 事件源的定义，参考 [CronSpec]({{<ref "cron-spec.md#cronspec" >}}) |
| **sink** *[SinkSpec](#sinkspec)*                             | *（可选）* 事件源关联的 Sink（可寻址的访问资源，即同步请求）定义，参考 [SinkSpec](#sinkspec) |

### SinkSpec

*从属 [EventSourceSpec](#eventsourcespec)*

| 字段                              | 描述                         |
| --------------------------------- | ---------------------------- |
| **ref** *[Reference](#reference)* | 参考 [Reference](#reference) |

### Reference

*从属 [SinkSpec](#sinkspec)*

> 引用资源一般为 [Knative Service](https://knative.dev/docs/reference/api/serving-api/#serving.knative.dev/v1.Service) 

| 字段                    | 描述                                                    |
| ----------------------- | ------------------------------------------------------- |
| **kind** *string*       | 引用资源的类型，默认为：`Service`                       |
| **namespace** *string*  | 引用资源的命名空间，默认与 Trigger 的命名空间一致       |
| **name** *string*       | 引用资源的名称，如：`function-ksvc`                     |
| **apiVersion** *string* | 引用资源的 apiVersion，默认为：`serving.knative.dev/v1` |

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

