---
title: "Function Signatures"
linkTitle: "Function Signatures"
weight: 3300
description: 
---

## Comparison of different function signatures

There're three function signatures in OpenFunction: `HTTP Functions`, `CloudEvent Functions`, and `OpenFunction Functions`. Let's explain this in more detail using Go function signature as an example.

`HTTP Functions` and `CloudEvent Functions` can be used to create sync functions while `OpenFunction Functions` can be used to create both sync and async functions. 

Further more `OpenFunction Functions` can utilize various Dapr building blocks including Bindings, Pub/sub etc. to create more powerful functions. (Dapr State management, Configuration will be supported soon)

|           | HTTP Functions | CloudEvent Functions | OpenFunction Functions |
|-----------|----------------|----------------------|------------------------|
| Signature | `func(http.ResponseWriter, *http.Request) error` | `func(context.Context, cloudevents.Event) error` | `func(ofctx.Context, []byte) (ofctx.Out, error)` |
| Sync Functions | Supported | Supported | Supported |
| Async Functions | Not supported | Not supported | Supported |
| Dapr Binding | Not supported | Not supported | Supported |
| Dapr Pub/sub | Not supported | Not supported | Supported |

## Samples

As you can see, `OpenFunction Functions` is the recommended function signature, and we're working on supporting this function signature in more language runtimes.

|           | HTTP Functions | CloudEvent Functions | OpenFunction Functions |
|-----------|----------------|----------------------|------------------------|
| go        | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-go), [Multi-functions](https://github.com/OpenFunction/samples/tree/main/functions/knative/multiple-functions-go), [log processing](https://github.com/OpenFunction/samples/blob/main/functions/knative/logs-handler-function/LogsHandler.go) |  | [Sync function with output binding](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding), [Kafka input & HTTP output binding](https://github.com/OpenFunction/samples/tree/main/functions/async/logs-handler-function), [Cron input & Kafka output binding](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input-kafka-output), [Cron input binding](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/cron-input), [Kafka input binding](https://github.com/OpenFunction/samples/tree/main/functions/async/bindings/kafka-input), [Kafka pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub) |
| Nodejs    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-node) |  | [Sync function with output binding](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding-node), [MQTT binding & pubsub](https://github.com/OpenFunction/samples/tree/main/functions/async/mqtt-io-node) |
| Python    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-python) |  |  |
| Java      | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-java) |  |  |
| DotNet    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-dotnet) |  |  |