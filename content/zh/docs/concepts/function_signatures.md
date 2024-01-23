---
title: "函数签名"
linkTitle: "函数签名"
weight: 3300
description:
---

## 不同函数签名的比较

OpenFunction中有三种函数签名：`HTTP`，`CloudEvent`和`OpenFunction`。让我们使用Go函数为例，详细解释一下。

`HTTP`和`CloudEvent`签名可以用来创建同步函数，而`OpenFunction`签名可以用来创建同步和异步函数。

此外，`OpenFunction`签名可以利用各种Dapr构建块，包括Bindings，Pub/sub等访问各种BaaS服务，帮助创建更强大的函数。（Dapr状态管理，配置将很快得到支持）

|           | HTTP | CloudEvent | OpenFunction |
|-----------|----------------|----------------------|------------------------|
| 签名 | `func(http.ResponseWriter, *http.Request) error` | `func(context.Context, cloudevents.Event) error` | `func(ofctx.Context, []byte) (ofctx.Out, error)` |
| 同步函数 | 支持 | 支持 | 支持 |
| 异步函数 | 不支持 | 不支持 | 支持 |
| Dapr Binding | 不支持 | 不支持 | 支持 |
| Dapr Pub/sub | 不支持 | 不支持 | 支持 |

## 示例

如您所见，`OpenFunction`签名是推荐的函数签名，我们正在努力在更多语言运行时支持此函数签名。

|           | HTTP | CloudEvent | OpenFunction |
|-----------|----------------|----------------------|------------------------|
| Go        | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-go), [多函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/multiple-functions-go), [带路径参数的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/path-parameters-function-go), [日志处理](https://github.com/OpenFunction/samples/blob/main/functions/knative/logs-handler-function/LogsHandler.go) | [带路径参数的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/path-parameters-function-go) | [带路径参数的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/path-parameters-function-go), [带输出绑定的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding), [Kafka输入 & HTTP输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/logs-handler-function), [Cron输入 & Kafka输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input-kafka-output), [Cron输入绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input), [Kafka输入绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/kafka-input), [Kafka pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub) |
| Nodejs    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-node) |  | [带输出绑定的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding-node), [MQTT绑定 & pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/mqtt-io-node) |
| Python    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-python) |  |  |
| Java      | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/java/hello-world) | [CloudEvent](https://github.com/OpenFunction/samples/tree/main/functions/knative/java/cloudevent) | [带输出的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/java/with-output-binding), [Cron输入 & Kafka输出绑定](https://github.com/OpenFunction/samples/tree/main/functions/async/java/cron-input-kafka-output), [Kafka pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/java/pubsub) |
| DotNet    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-dotnet) |  |  |