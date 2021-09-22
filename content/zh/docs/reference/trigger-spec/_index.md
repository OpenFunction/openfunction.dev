---
title: "Trigger 参数说明"
linkTitle: "Trigger 参数说明"
weight: 20
description: >
    Trigger CRD 参数规范说明
---

## Trigger

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | events.openfunction.io/v1alpha1                              |
| **kind** *string*                                            | Trigger                                                      |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *（可选）* 参考 [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) 文档 |
| **spec** *[TriggerSpec](#triggerspec)*                       | 事件触发器的规格，参考 [TriggerSpec](#triggerspec)           |
| **status** *TriggerStatus*                                   | 触发器的状态                                                 |

### TriggerSpec

*从属 [Trigger](#trigger)*

| 字段                                           | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| **eventBus** *string*                          | *（可选）* 事件触发器关联的 EventBus 资源名称                |
| **inputs** *map\[string][Input](#input)*       | *（可选）* 输入触发器的事件，参考 map\[string][Input](#input) |
| **subscribers** *\[][Subscriber](#subscriber)* | *（可选）* 触发器的订阅者，参考 \[][Subscriber](#subscriber) |

### Input

*从属 [TriggerSpec](#triggerspec)*

| 字段                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **namespace** *string*   | *（可选）* 事件源的的命名空间名称，默认与 Trigger 的命名空间一致。如：`default` |
| **eventSource** *string* | 事件源名称，如：`kafka-eventsource`                          |
| **event** *string*       | 事件名称，如：`eventA`                                       |

### Subscriber

*从属 [TriggerSpec](#triggerspec)*

| 字段                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| **condition** *string*                     | 触发器的触发条件，参考 [cel-spec](https://github.com/google/cel-spec/blob/master/doc/langdef.md) 获取更多写法规范。如：`eventA && eventB`，`eventA || eventB` |
| **sink** *[SinkSpec](#sinkspec)*           | *（可选）* 触发的 Sink（可寻址的访问资源，即同步请求）定义，参考 [SinkSpec](#sinkspec) |
| **deadLetterSink** *[SinkSpec](#sinkspec)* | *（可选）* 触发的死信 Sink（可寻址的访问资源，即同步请求）定义，参考 [SinkSpec](#sinkspec) |
| **topic** *string*                         | *（可选）* 触发的发送给事件总线的 topic 名称，如：`topicTriggered` |
| **deadLetterTopic** *string*               | *（可选）* 触发的发送给事件总线的死信 topic 名称，如：`topicDL` |

### SinkSpec

*从属 [Subscriber](#subscriber)*

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

