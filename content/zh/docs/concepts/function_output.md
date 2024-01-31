---
title: "函数输出"
linkTitle: "函数输出"
weight: 3210
description:
---

## 函数输出

输出是函数可以发送数据的组件，包括：

- 任何 [Dapr 输出绑定组件](https://docs.dapr.io/reference/components-reference/supported-bindings/) 的 [Dapr 绑定构建块](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/)
- 任何 [Dapr Pub/sub brokers 组件](https://docs.dapr.io/reference/components-reference/supported-pubsub/) 的 [Dapr Pub/sub 构建块](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)

例如，[这里](https://github.com/OpenFunction/samples/blob/main/functions/async/bindings/cron-input-kafka-output) 你可以找到一个带有 cron 输入绑定和 Kafka 输出绑定的异步函数：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: cron-input-kafka-output
spec:
  ...
  serving:
    ...
    outputs:
      - dapr:
          name: kafka-server
          type: bindings.kafka
          operation: "create"
    bindings:
      kafka-server:
        type: bindings.kafka
        version: v1
        metadata:
        - name: brokers
          value: "kafka-server-kafka-brokers:9092"
        - name: topics
          value: "sample-topic"
        - name: consumerGroup
          value: "bindings-with-output"
        - name: publishTopic
          value: "sample-topic"
        - name: authRequired
          value: "false"
```

[这里](https://github.com/OpenFunction/samples/blob/main/functions/async/bindings/cron-input-kafka-output) 是另一个使用 Kafka Pub/sub 组件作为输入的异步函数示例

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: autoscaling-subscriber
spec:
  ...
  serving:
    ...
    runtime: "async"
    outputs:
      - dapr:
          name: kafka-server
          type: pubsub.kafka
          topic: "sample-topic"
    pubsub:
      kafka-server:
        type: pubsub.kafka
        version: v1
        metadata:
          - name: brokers
            value: "kafka-server-kafka-brokers:9092"
          - name: authRequired
            value: "false"
          - name: allowedTopics
            value: "sample-topic"
          - name: consumerID
            value: "autoscaling-subscriber"
```
