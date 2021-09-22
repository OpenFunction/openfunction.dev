---
title: "Kafka 参数说明"
linkTitle: "Kafka 参数说明"
weight: 20
description: >
    Kafka 事件源参数说明
---

### KafkaSpec

*从属 [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> EventSource 会根据 KafkaSpec 生成用于适配 Kafka 事件源的 [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#component-format) ，原则上我们会尽量维持相关参数的一致性。你可以通过访问 [Kafka binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#spec-metadata-fields) 获取更多信息。

| 字段                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **brokers** *string*        | 以逗号分隔的 Kafka 服务器地址的字符串。如：`localhost:9092`  |
| **authRequired** *bool*     | 是否启用 Kafka 服务器的 [SASL](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) 认证。可选：`true`， `false` |
| **topic** *string*          | Kafka 事件源的 topic 名称，如：`topic`1，`myTopic`           |
| **saslUsername** *string*   | *（可选）* 用于认证的 SASL 用户名。只有在 authRequired 为 `true` 时才需要设置。如：`admin` |
| **saslPassword** *string*   | *（可选）* 用于认证的 SASL 用户密码。只有在 authRequired 为 `true` 时才需要设置。如：`123456` |
| **maxMessageBytes** *int64* | *（可选）* 单个消息允许包含的最大字节数。默认为 `1024`。如：`2048` |