---
title: "Nats Streaming 参数说明"
linkTitle: "Nats Streaming 参数说明"
weight: 15
description: >
    Nats Streaming 事件总线参数说明
---

### NatsStreamingSpec

*从属 [EventBusSpec]({{<ref "_index.md#eventbusspec" >}})*

> EventBus（ClusterEventBus） 会将配置提供给 EventSource 和 Trigger 引用，以便生成对应的 [Dapr Pub/Sub Components](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#component-format) 从事件总线处获取消息，原则上我们会尽量维持相关参数的一致性。你可以通过访问 [NATS Streaming pubsub spec](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#spec-metadata-fields) 获取更多信息。

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **natsURL** *string*                                         | NATS 服务器的地址，如：`nats://localhost:4222`               |
| **natsStreamingClusterID** *string*                          | NATS 集群 ID，如：`stan`                                     |
| **subscriptionType** *string*                                | 订阅器类型，可选：`topic`， `queue`                          |
| **ackWaitTime** *string*                                     | *（可选）* 请参考 [Acknowledgements](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) 。如：`300ms` |
| **maxInFlight** *int64*                                      | *（可选）* 请参考 [Max In Flight](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) 。如：`25` |
| **durableSubscriptionName** *string*                         | *（可选）* [持久化订阅器](https://docs.nats.io/developing-with-nats-streaming/durables) 的名称。如：`my-durable` |
| **deliverNew** *bool*                                        | *（可选）* 订阅器选项（只能使用一个）。是否只发送新消息。可选：`true`，`false` |
| **startAtSequence** *int64*                                  | *（可选）* 订阅器选项（只能使用一个）。设置起始序列位置和状态。如：`100000` |
| **startWithLastReceived** *bool*                             | *（可选）* 订阅器选项（只能使用一个）。是否将起始位置设置为最新消息处。可选：`true`，`false` |
| **deliverAll** *bool*                                        | *（可选）* 订阅器选项（只能使用一个）。是否发送所有可用的信息。可选：`true`，`false` |
| **startAtTimeDelta** *string*                                | *（可选）* 订阅器选项（只能使用一个）。使用差值形式设置所需的开始时间位置和状态。如：`10m`，`23s` |
| **startAtTime** *string*                                     | *（可选）* 订阅器选项（只能使用一个）。使用标值形式设置所需的开始时间位置和状态。如：`Feb 3, 2013 at 7:54pm (PST)` |
| **startAtTimeFormat** *string*                               | *（可选）* 必须与 startAtTime 一起使用。设置时间的格式。如：`Jan 2, 2006 at 3:04pm (MST)` |
| **scaleOption** [NatsStreamingScaleOption](#natsstreamingscaleoption) | *（可选）* Nats streaming 的自动伸缩选项                     |

### NatsStreamingScaleOption

*从属 [NatsStreamingSpec](#natsstreamingspec)*

| 字段                                                         | 描述                                                 |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| *[GenericScaleOption]({{< ref "../eventbus-spec#genericscaleoption" >}})* | 通用的自动伸缩配置                                   |
| **natsServerMonitoringEndpoint** *string*                    | Nats streaming 的指标监控端点                        |
| **queueGroup** *string*                                      | Nats streaming 的 queue group 名称                   |
| **durableName** *string*                                     | Nats streaming 的 durable 名称                       |
| **subject** *string*                                         | 伸缩器监控的 subject 名称，如：`topicA`, `myTopic`   |
| **lagThreshold** *string*                                    | 伸缩器的触发阈值，此处为 Nats Streaming 的消息延迟量 |

