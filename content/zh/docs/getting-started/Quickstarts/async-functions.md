---
title: "创建异步函数"
linkTitle: "创建异步函数"
weight: 2230
description:
---

> 在创建任何函数之前，请确保您已安装所有[先决条件](../prerequisites)

异步函数是事件驱动的，它们的输入通常是来自非HTTP事件源的事件，如消息队列、cron触发器、MQTT代理等，通常客户端在通过传递事件触发异步函数后不会等待立即的响应。以下是不同语言的一些异步函数示例：

|           | 异步函数 |
|-----------|-----------------|
| Go        | [Kafka输入和HTTP输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/logs-handler-function), [Cron输入和Kafka输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input-kafka-output), [Cron输入绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input), [Kafka输入绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/kafka-input), [Kafka pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub) |
| Nodejs    | [MQTT绑定和pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/mqtt-io-node) |
| Python    |  |
| Java      | [Cron输入和Kafka输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/java/cron-input-kafka-output), [Kafka pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/java/pubsub) |
| DotNet    |  |

> 您可以在[这里](../../../concepts/function_signatures/#samples)找到更多函数示例
