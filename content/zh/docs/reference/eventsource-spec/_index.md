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