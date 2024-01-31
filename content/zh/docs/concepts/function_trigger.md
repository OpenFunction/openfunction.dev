---
title: "函数触发器"
linkTitle: "函数触发器"
weight: 3200
description:
---
函数 `触发器` 用于定义如何触发函数。目前有两种触发器：`HTTP 触发器` 和 `Dapr 触发器`。默认的触发器是 `HTTP 触发器`。

## HTTP 触发器

`HTTP 触发器` 通过 HTTP 请求触发函数。你可以像这样为函数定义一个 `HTTP 触发器`：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    triggers:
      http:
        port: 8080
        route:
          rules:
            - matches:
                - path:
                    type: PathPrefix
                    value: /echo
```

## Dapr 触发器

`Dapr 触发器` 通过 `Dapr bindings` 或 `Dapr pubsub` 的事件触发函数。你可以像这样定义一个带有 `Dapr 触发器` 的函数：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: logs-async-handler
  namespace: default
spec:
  serving:
    bindings:
      kafka-receiver:
        metadata:
          - name: brokers
            value: kafka-server-kafka-brokers:9092
          - name: authRequired
            value: "false"
          - name: publishTopic
            value: logs
          - name: topics
            value: logs
          - name: consumerGroup
            value: logs-handler
        type: bindings.kafka
        version: v1
    triggers:
      dapr:
        - name: kafka-receiver
          type: bindings.kafka
```

## 函数输入

`输入` 是函数可以从中获取额外输入数据的地方，目前支持 `Dapr 状态存储` 作为 `输入`。

你可以像这样定义函数输入：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: logs-async-handler
  namespace: default
spec:
  serving:
    triggers:
      inputs:
        - dapr:
            name: mysql
            type: state.mysql
```