---
title: "Kafka"
linkTitle: "Kafka"
weight: 20
description: >
    Event source specification of Kafka
---

### KafkaSpec

*Belong to [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> The EventSource generates [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#component-format) for adapting Kafka event sources according to the KafkaSpec, and in principle we try to maintain the consistency of the relevant parameters. You can get more information by visiting the [Kafka binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#spec-metadata-fields).

| Field                                                 | Description                                                  |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| **brokers** *string*                                  | A comma-separated string of Kafka server addresses. E.g. `localhost:9092` |
| **authRequired** *bool*                               | Whether to enable [SASL](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) authentication for the Kafka server. Optional: `true`, `false` |
| **topic** *string*                                    | The topic name of the Kafka event source, e.g. `topicA`, `myTopic` |
| **saslUsername** *string*                             | *(Optional)* The SASL username to use for authentication. Only required if authRequired is `true`. For example: `admin` |
| **saslPassword** *string*                             | *(Optional)* The SASL user password for authentication. Only required if authRequired is `true`. For example: `123456` |
| **maxMessageBytes** *int64*                           | *(Optional)* The maximum number of bytes a single message is allowed to contain. Default is `1024`. For example: `2048` |
| **scaleOption** [KafkaScaleOption](#kafkascaleoption) | *(Optional)* Kafka's scale configuration                     |

### KafkaScaleOption

*Belong to [KafkaSpec](#kafkaspec)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| *[GenericScaleOption]({{< ref "../eventsource-spec#genericscaleoption" >}})* | Generic scale configuration                                  |
| **consumerGroup** *string*                                   | Kafka's consumer group name                                  |
| **topic** *string*                                           | Topic under monitoring, e.g. `topicA`, `myTopic`             |
| **lagThreshold** *string*                                    | Threshold for triggering scaling, in this case is the Kafka's lag |

