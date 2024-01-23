---
title: "创建同步函数"
linkTitle: "创建同步函数"
weight: 2220
description:
---

> 在创建任何函数之前，请确保您已安装所有的[先决条件](../prerequisites)

同步函数是其输入为HTTP请求的有效载荷的函数，输出或响应在函数逻辑处理输入有效载荷后立即发送给等待的客户端。以下是不同语言的一些同步函数示例：

|           | 同步函数 |
|-----------|----------------|
| Go        | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-go), [多函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/multiple-functions-go), [带路径参数的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/path-parameters-function-go), [日志处理](https://github.com/OpenFunction/samples/blob/main/functions/knative/logs-handler-function/LogsHandler.go), [带输出绑定的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding) |
| Nodejs    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-node), [带输出绑定的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding-node) |
| Python    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-python) |
| Java      | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/java/hello-world), [带输出的同步函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/java/with-output-binding) |
| DotNet    | [Hello World](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-dotnet) |

> 您可以在[这里](../../../concepts/function_signatures/#samples)找到更多函数示例
