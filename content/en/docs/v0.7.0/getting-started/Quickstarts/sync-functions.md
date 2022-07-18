---
title: "Create sync functions"
linkTitle: "Create sync functions"
weight: 2220
description: 
---

> Before you creating any functions, make sure you've installed all the [prerequisites](../prerequisites)

Sync functions are funtions whose inputs are payloads of HTTP requests, and the output or response are sent to the waiting client immediately after the function logic finishes processing the inputs payload. Below you can find some sync function examples in different languages:

|           | Sync Functions |
|-----------|----------------|
| Go        | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-go), [Multi-functions](https://github.com/OpenFunction/samples/tree/main/functions/knative/multiple-functions-go), [log processing](https://github.com/OpenFunction/samples/blob/main/functions/knative/logs-handler-function/LogsHandler.go), [Sync function with output binding](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding) |
| Nodejs    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-node), [Sync function with output binding](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding-node) |
| Python    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-python) |
| Java      | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-java) |
| DotNet    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-dotnet) |

> You can find more function samples [here](../../../concepts/function_signatures/#samples)