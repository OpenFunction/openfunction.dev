---
title: "Function Inputs and Outputs"
linkTitle: "Function Inputs and Outputs"
weight: 3200
description: 
---
Functions usually have inputs and outputs.

## Function Inputs

For a sync function, the input is always the payload of the HTTP request.

For an async function, the input data comes from:
- Any [Dapr Input Binding components](https://docs.dapr.io/reference/components-reference/supported-bindings/) of the [Dapr Bindings Building Block](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/)
- Any [Dapr Pub/sub brokers components](https://docs.dapr.io/reference/components-reference/supported-pubsub/) of the [Dapr Pub/sub Building Block](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)

## Function Outputs

For a sync function, the output can be sent through the HTTP response.

Both sync functions and async functions can send outputs to Dapr components including:
- Any [Dapr Output Binding components](https://docs.dapr.io/reference/components-reference/supported-bindings/) of the [Dapr Bindings Building Block](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/)
- Any [Dapr Pub/sub brokers components](https://docs.dapr.io/reference/components-reference/supported-pubsub/) of the [Dapr Pub/sub Building Block](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)

For example, [here](https://github.com/OpenFunction/samples/blob/main/functions/async/bindings/cron-input-kafka-output) you can find an async function with a cron input binding and a Kafka output binding:

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: cron-input-kafka-output
spec:
  ...
  serving:
    ...
    runtime: "async"
    inputs:
      - name: cron
        component: cron
    outputs:
      - name: sample
        component: kafka-server
        operation: "create"
    bindings:
      cron:
        type: bindings.cron
        version: v1
        metadata:
        - name: schedule
          value: "@every 2s"
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
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: autoscaling-subscriber
spec:
  ...
  serving:
    ...
    runtime: "async"
    inputs:
      - name: producer
        component: kafka-server
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

Sync functions can also send output to Dapr output binding components or Pub/sub components, [here](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding) is an example:

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-front
spec:
  serving:
    ...
    runtime: knative
    outputs:
      - name: target
        component: kafka-server
        operation: "create"
    bindings:
      kafka-server:
        type: bindings.kafka
        version: v1
        metadata:
          - name: brokers
            value: "kafka-server-kafka-brokers:9092"
          - name: authRequired
            value: "false"
          - name: publishTopic
            value: "sample-topic"
          - name: topics
            value: "sample-topic"
          - name: consumerGroup
            value: "function-front"
```
