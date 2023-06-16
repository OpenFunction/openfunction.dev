---
title: "OpenFunction v1.1.0 发布：支持 Dapr 状态管理，重构函数触发器"
linkTitle: "Release v1.1.0"
date: 2023-06-12
weight: 92
---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) 是一个开源的云原生 FaaS（Function as a Service，函数即服务）平台，旨在帮助开发者专注于业务逻辑的研发。在过去的几个月里，OpenFunction 社区一直在努力工作，为 OpenFunction 1.1.0 版本的发布做准备。今天，我们非常高兴地宣布 OpenFunction 1.1.0 已经发布了！感谢社区各位小伙伴的贡献和反馈！

OpenFunction 1.1.0 版本带来了两个新的功能：新增 v1beta2 API，支持 Dapr 状态管理。此外，该版本还有多项强化（重构函数触发器等）及 bug 修复，使 OpenFunction 更加稳定和易用。

以下是本次版本更新的主要内容：

## 新增 v1beta2 API

在此版本中，我们新增了 v1beta2 API，原 v1beta1 API 已弃用，将来会被删除。v1beta2 中有不少重构，你可以在这个 [proposal](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/20230330-support-dapr-state-management.md) 中了解更多细节。

## 支持 Dapr 状态管理

之前，OpenFunction 支持 Dapr 发布/订阅和绑定构建块，状态管理也是有用的构建块之一，它对于具有状态的函数非常有用。使用状态存储组件，您可以构建具有持久状态的函数，这些函数可以保存和恢复它们的状态。

现在你可以在 Function CR 中定义状态存储，OpenFunction 将管理相应的 Dapr 组件。

你的函数可以使用简单封装的 Dapr 的状态管理 API 来保存、读取和查询定义的状态存储中的键/值对。

## 重构函数触发器

之前，我们使用 `runtime: knative` 和 `runtime: async` 来区分同步和异步函数，这会增加学习曲线。实际上，同步和异步函数之间的区别在于触发类型:

- 同步函数由 HTTP 事件触发，这可以通过指定 `runtime: knative` 来定义。
- 异步函数由 Dapr 绑定组件或 Dapr 发布者事件触发。要指定异步函数的触发器，必须同时使用 `runtime: async` 和 `inputs`。

因此，我们可以使用 `triggers` 来替代 `runtime` 和 `inputs`。

`HTTP Trigger` 通过 HTTP 请求触发一个函数。您可以这样定义一个 `HTTP Trigger`：

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

`Dapr Trigger` 使用 `Dapr bindings` 或 `Dapr pubsub` 的事件触发一个函数。你可以用 `Dapr Trigger` 定义一个函数：

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

## 其他改进和优化

- 从网关状态中删除 lastTransitionTime 字段，以防止频繁触发 reconcile。
- 允许在创建 Dapr 组件时设置作用域。
- 使用 OpenFunction 策略时，支持设置缓存镜像以提高构建性能。
- 支持设置 OpenFunction 构建策略的 bash 镜像。

以上就是 OpenFunction v1.1.0 的主要功能变化，在此十分感谢各位贡献者的参与和贡献。如果您正在寻找一款高效、灵活的云原生函数开发平台，那么 OpenFunction v1.1.0 绝对不容错过。

了解更多关于 OpenFunction 和本次版本更新的信息，欢迎访问我们的官方网站和 Github 页面。

- [官网](https://openfunction.dev/)：https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.1.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.1.0