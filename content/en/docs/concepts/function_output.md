---
title: "Function Outputs"
linkTitle: "Function Outputs"
weight: 3210
description: 
---

## Function Outputs

Output is a component that the function can send data to, include:

- Any [Dapr Output Binding components](https://docs.dapr.io/reference/components-reference/supported-bindings/) of the [Dapr Bindings Building Block](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/)
- Any [Dapr Pub/sub brokers components](https://docs.dapr.io/reference/components-reference/supported-pubsub/) of the [Dapr Pub/sub Building Block](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)

For example, [here](https://github.com/OpenFunction/samples/blob/main/functions/async/bindings/cron-input-kafka-output) you can find an async function with a cron input binding and a Kafka output binding:

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

[Here](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub/subscriber) is another async function example that use a Kafka Pub/sub component as input.

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
