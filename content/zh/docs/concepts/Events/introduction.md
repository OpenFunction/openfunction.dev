---
title: "简介"
linkTitle: "简介"
weight: 3400
description:
---

## 概述

OpenFunction 事件是 OpenFunction 的事件管理框架。它提供以下核心特性：

- 支持通过同步和异步调用触发目标函数
- 用户定义的触发判断逻辑
- OpenFunction 事件的组件可以由 OpenFunction 本身驱动

## 架构

以下图示说明了 OpenFunction 事件的架构。

![openfunction-events](/images/docs/en/concepts/events/openfunction-events.svg)

## 概念

### EventSource

EventSource 定义了事件的生产者，例如 Kafka 服务，对象存储服务，甚至是函数。它包含了这些事件生产者的描述和发送这些事件的信息。

EventSource 支持以下类型的事件源服务器：

- Kafka
- Cron（调度器）
- Redis

### EventBus（ClusterEventBus）

EventBus 负责聚合事件并使其持久化。它包含了一个通常是消息队列（如 NATS Streaming 和 Kafka）的事件总线 broker 的描述，并为 EventSource 和 Trigger 提供这些配置。

EventBus 默认处理命名空间范围内的事件总线适配。对于集群范围，ClusterEventBus 可作为事件总线适配器，并在其他组件无法找到命名空间下的 EventBus 时生效。

EventBus 支持以下事件总线 broker：

- NATS Streaming

### Trigger

Trigger 是事件目的的抽象，例如接收到消息时需要做什么。它包含了由您定义的事件的目的，这告诉触发器它应该从哪个 EventSource 获取事件，并根据给定的条件决定是否触发目标函数。

## 参考

有关更多信息，请参阅 [EventSource 规范](https://openfunction.dev/docs/reference/component-reference/eventsource-spec/eventsource-spec) 和 [EventBus 规范](https://openfunction.dev/docs/reference/component-reference/eventbus-spec/eventbus-spec)。