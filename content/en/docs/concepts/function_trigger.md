---
title: "Function Trigger"
linkTitle: "Function Trigger"
weight: 3200
description: 
---
Function `Triggers` are used to define how to trigger a function. Currently, there are two kinds of triggers: `HTTP Trigger`, and `Dapr Trigger`. The default trigger is `HTTP trigger`.

## HTTP Trigger

`HTTP Trigger` triggers a function with an HTTP request. You can define an `HTTP Trigger` for a function like this:

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

## Dapr Trigger

`Dapr Trigger` triggers a function with events from `Dapr bindings` or `Dapr pubsub`. You can define a function with `Dapr Trigger` like this:


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

## Function Inputs

`Input` is where a function can get extra input data from, `Dapr State Stores` is supported as `Input` currently.

You can define function input like this:

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