---
title: "BaaS Integration"
linkTitle: "BaaS Integration"
weight: 3350
description: 
---

One of the unique features of OpenFunction is its simple integration with various backend services (BaaS) through [Dapr](https://docs.dapr.io/concepts/components-concept/). Currently, OpenFunction supports Dapr [pub/sub](https://docs.dapr.io/reference/components-reference/supported-pubsub/) and [bindings](https://docs.dapr.io/reference/components-reference/supported-bindings/) building blocks, and more will be added in the future.

In OpenFunction v0.7.0 and versions prior to v0.7.0, OpenFunction integrates with BaaS by injecting a dapr sidecar container into each function instance's pod, which leads to the following problems：

- The entire function instance's launch time is slowed down by the launching of the dapr sidecar container.
- The dapr sidecar container may consume more resources than the function container itself.

To address the problems above, OpenFunction introduces the `Dapr Standalone Mode` in v0.8.0.

## Dapr Standalone Mode

In Dapr standalone mode, one `Dapr Proxy service` will be created for each function which is then shared by all instances of this function. This way, there is no need to launch a seperate Dapr sidecar container for each function instance anymore which reduces the function launching time significantly.

<div align=center><img width="100%" height="100%" src=/images/docs/en/concepts/function/dapr_proxy.png/></div>

## Choose the appropriate Dapr Service Mode

So now you've 2 options to integrate with BaaS:

- `Dapr Sidecar Mode`
- `Dapr Standalone Mode`

You can choose the appropriate Dapr Service Mode for your functions. The `Dapr Standalone Mode` is the recommened and default mode. You can use `Dapr Sidecar Mode` if your function doesn't scale frequently or you've difficulty to use the `Dapr Standalone Mode`.

You can control how to integrate with BaaS with 2 flags, both can be set in function's `spec.serving.annotations`:

- `openfunction.io/enable-dapr` can be set to `true` or `false`
- `openfunction.io/dapr-service-mode` can be set to `standalone` or `sidecar`
- When `openfunction.io/enable-dapr` is set to `true`, users can choose the `Dapr Service Mode` by setting `openfunction.io/dapr-service-mode` to `standalone` or `sidecar`.
- When `openfunction.io/enable-dapr` is set to `false`, the value of `openfunction.io/dapr-service-mode` will be ignored and neither `Dapr Sidecar` nor `Dapr Proxy Service` will be launched. 

There're default values for both of these two flags if they're not set.

- The value of `openfunction.io/enable-dapr` will be set to `true` if it's not defined  in `spec.serving.annotations` and the function definition contains either `spec.serving.inputs` or `spec.serving.outputs`. Otherwise it will be set to `false`.
- The default value of `openfunction.io/dapr-service-mode` is `standalone` if not set.

Below you can find a function example to set these two flags:

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: cron-input-kafka-output
spec:
  version: "v2.0.0"
  image: openfunctiondev/cron-input-kafka-output:v1
  imageCredentials:
    name: push-secret
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "HandleCronInput"
      FUNC_CLEAR_SOURCE: "true"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/async/bindings/cron-input-kafka-output"
      revision: "main"
  serving:
    annotations:
      openfunction.io/enable-dapr: "true"
      openfunction.io/dapr-service-mode: "standalone"
    template:
      containers:
        - name: function # DO NOT change this
          imagePullPolicy: IfNotPresent 
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