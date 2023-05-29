---
title: "Function Trigger"
linkTitle: "Function Trigger"
weight: 3200
description: 
---
Function triggers used to triggered the function. Now there are two kind of triggers, HTTP trigger and dapr trigger.

## HTTP Trigger

HTTP trigger triggered the function with a HTTP request. To define a HTTP trigger, you can follow this:

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

To define a dapr trigger, you can follow this:

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

Function input is a component that function can get data from, now only support dapr state store.

To define a input, you can follow this:

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