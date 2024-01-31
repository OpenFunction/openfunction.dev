---
title: "BaaS 集成"
linkTitle: "BaaS 集成"
weight: 3350
description:
---

OpenFunction 的一个独特特性是它可以通过 [Dapr](https://docs.dapr.io/concepts/components-concept/) 简单地集成各种后端服务 (BaaS)。目前，OpenFunction 支持 Dapr 的 [pub/sub](https://docs.dapr.io/reference/components-reference/supported-pubsub/) 和 [bindings](https://docs.dapr.io/reference/components-reference/supported-bindings/) 构建块，未来将会添加更多。

在 OpenFunction v0.7.0 及之前的版本中，OpenFunction 通过在每个函数实例的 pod 中注入一个 dapr sidecar 容器来集成 BaaS，这导致了以下问题：

- dapr sidecar 容器的启动拖慢了整个函数实例的启动时间。
- dapr sidecar 容器可能消耗的资源比函数容器本身还要多。

为了解决上述问题，OpenFunction 在 v0.8.0 中引入了 `Dapr 独立模式`。

## Dapr 独立模式

在 Dapr 独立模式中，每个函数都会创建一个 `Dapr 代理服务`，然后由该函数的所有实例共享。这样，就不再需要为每个函数实例启动一个单独的 Dapr sidecar 容器，从而大大减少了函数的启动时间。

<div align=center><img width="100%" height="100%" src=/images/docs/en/concepts/function/dapr_proxy.png/></div>

## 选择适当的 Dapr 服务模式

所以现在你有两个选项来集成 BaaS：

- `Dapr Sidecar 模式`
- `Dapr 独立模式`

你可以为你的函数选择适当的 Dapr 服务模式。`Dapr 独立模式` 是推荐的和默认的模式。如果你的函数不经常扩展，或者你难以使用 `Dapr 独立模式`，你可以使用 `Dapr Sidecar 模式`。

你可以通过设置两个标志来控制如何集成 BaaS，这两个标志都可以在函数的 `spec.serving.annotations` 中设置：

- `openfunction.io/enable-dapr` 可以设置为 `true` 或 `false`
- `openfunction.io/dapr-service-mode` 可以设置为 `standalone` 或 `sidecar`
- 当 `openfunction.io/enable-dapr` 设置为 `true` 时，用户可以通过将 `openfunction.io/dapr-service-mode` 设置为 `standalone` 或 `sidecar` 来选择 `Dapr 服务模式`。
- 当 `openfunction.io/enable-dapr` 设置为 `false` 时，`openfunction.io/dapr-service-mode` 的值将被忽略，既不会启动 `Dapr Sidecar`，也不会启动 `Dapr 代理服务`。

如果这两个标志没有设置，它们都有默认值。

- 如果 `openfunction.io/enable-dapr` 在 `spec.serving.annotations` 中没有定义，并且函数定义包含 `spec.serving.inputs` 或 `spec.serving.outputs`，那么 `openfunction.io/enable-dapr` 的值将被设置为 `true`。否则，它将被设置为 `false`。
- 如果没有设置 `openfunction.io/dapr-service-mode`，其默认值为 `standalone`。

下面你可以找到一个设置这两个标志的函数示例：

```yaml
apiVersion: core.openfunction.io/v1beta2
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
    triggers:
      dapr:
        - name: cron
          type: bindings.cron
    outputs:
      - dapr:
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