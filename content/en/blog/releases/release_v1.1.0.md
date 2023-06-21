---
title: "Announcing OpenFunction 1.1.0: Support Dapr State Management and Refactor Function Triggers"
linkTitle: "Release v1.1.0"
date: 2023-06-12
weight: 92

---

OpenFunction is a cloud-native open-source FaaS (Function as a Service) platform aiming to let you focus on your business logic only. Today, we are thrilled to announce the general availability of OpenFunction 1.1.0.

In this release, we have added the v1beta2 API and support Dapr State management. In addition, we enhanced some features and fixed bugs, making OpenFunction more stable and easy to use.

The following introduces the major updates.

## Add the v1beta2 API

In this release, we have added the v1beta2 API. The v1beta1 API has been deprecated and will be removed. You can learn more about the v1beta2 API from this [proposal](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/20230330-support-dapr-state-management.md).

### Support Dapr state management

Previously, OpenFunction supports the pub/sub and bindings building blocks, and state management is a building block that is useful for stateful functions. With the use of state store components, you can build functions with persistent state, allowing them to save and restore their own states.

You can define state stores in Function CR, and OpenFunction will manage the corresponding Dapr components.

The functions can use the encapsulated state management API of Dapr to save, read, and query key/value pairs in the defined state storage.

## Refactor function triggers

Previously, we use `runtime: knative` and `runtime: async` to distinguish sync and async functions, which is sort of difficult. Actually the difference between sync and async functions lies in the trigger type:

- Sync functions are triggered by `HTTP` events, which are defined by specifying `runtime: knative`.

- Async functions are triggered by events from components of `Dapr bindings` or `Dapr pubsub`. `runtime: async` and `inputs` have to be used together to specify triggers for async functions.

Now we use `triggers` to replace `runtime` and `inputs`.

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

## Other enhancements

- Delete the `lastTransitionTime` field from the gateway status to prevent frequent triggering of reconcile.
- Allow to set scopes when creating Dapr components.
- Support the ability to set cache images to improve build performance when using OpenFunction strategies.
- Support the ability to set bash images of OpenFunction strategies.

These are the main feature changes in OpenFunction v1.1.0 and we would like to thank all contributors for your contributions. If you are looking for an efficient and flexible cloud-native function development platform, OpenFunction v1.1.0 is the perfect choice for you.

For more details and documentation, please visit our website and GitHub repo.

- [Official Website](https://openfunction.dev/): https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.1.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.1.0