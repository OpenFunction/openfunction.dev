---
title: "使用 SkyWalking 为 OpenFunction 提供可观测能力"
linkTitle: "使用 SkyWalking 为 OpenFunction 提供可观测能力"
weight: 5300
description: >	
    本文介绍了如何使用 SkyWalking 为 OpenFunction 构建可观测性解决方案。
---

## 功能概览

尽管 FaaS 允许开发者专注于他们的业务代码而不用担心底层的实现，但对函数服务进行功能调试和故障排除是很困难的。因此，OpenFunction 设法引入了可观测性能力来提高其可用性和稳定性。

SkyWalking 提供了在许多不同场景下观测和监控分布式系统的解决方案。OpenFunction 将 [go2sky](https://github.com/SkyAPM/go2sky)（SkyWalking 的 Go 语言代理）捆绑在 OpenFunction 追踪器选项中，以提供分布式追踪、函数性能统计和函数依赖关系图。

## 先决条件

- 您已安装了 [OpenFunction](../../getting-started/installation/)。
- 您已创建了一个 [Kubernetes Secret](../../getting-started/your-first-function/function-go/#create-a-secret)。
- 您已安装了 [SkyWalking v9](https://github.com/apache/skywalking-kubernetes#apache-skywalking-kubernetes)。
- 您已创建了一个 [Kafka 服务和主题](../interact-with-dapr-output-binding/#create-a-kafka-server-and-topic)。

## 追踪功能的可配置参数

下表描述了 OpenFunction 中目前可用的追踪参数。

| 名称                 | 描述                                                          | 示例                   |
| -------------------- | ------------------------------------------------------------- | ---------------------- |
| `enabled`            | 使能追踪功能，默认为 `false`                                  | `true`, `false`        |
| `provider.name`      | 可以设置为 `skywalking`、`opentelemetry`（待定）              | `skywalking`           |
| `provider.oapServer` | SkyWalking OAP 服务器地址                                     | `skywalking-opa:11800` |
| `tags`               | 一组键值对，用于追踪中的追踪所使用的 Span 自定义标签          |                        |
| `tags.func`          | 函数的名称，该值将被自动填充                                  | `function-a`           |
| `tags.layer`         | 表示被追踪的服务类型，当你使用该函数时，它应该被设置为 `faas` | `faas`                 |
| `baggage`            | 一个键值对的集合，存在于追踪中，也需要跨进程边界传输          |                        |

下面是一个 JSON 格式的配置参考，您可以基于此了解追踪配置的大致数据格式。

```json
{
  "enabled": true,
  "provider": {
    "name": "skywalking",
    "oapServer": "skywalking-oap:11800"
  },
  "tags": {
    "func": "function-a",
    "layer": "faas",
    "tag1": "value1",
    "tag2": "value2"
  },
  "baggage": {
    "key": "key1",
    "value": "value1"
  }
}
```

## 启用 OpenFunction 的追踪功能

### 选项一：全局配置

下文使用 `skywalking-oap.default:11800` 作为集群中 `skywalking-oap` 服务的样例地址。

1. 运行下面的命令，修改 `openfunction` 命名空间中的 ConfigMap `openfunction-config`。

   ```shell
   kubectl edit configmap openfunction-config -n openfunction
   ```

2. 参照下面的例子修改 `data.plugins.tracing` 中的内容，并保存这个修改。

   ```yaml
   data:
     plugins.tracing: |
       enabled: true
       provider:
         name: "skywalking"
         oapServer: "skywalking-oap:11800"
       tags:
         func: tracing-function
         layer: faas
         tag1: value1
         tag2: value2
       baggage:
         key: "key1"
         value: "value1"
   ```

### 选项二：函数级别配置

要在函数级别启用跟踪配置，请在函数清单的 `metadata.annotations` 下添加 `plugins.tracing` 字段，如下所示。

```yaml
metadata:
  name: tracing-function
  annotations:
    plugins.tracing: |
      enabled: true
      provider:
        name: "skywalking"
        oapServer: "skywalking-oap:11800"
      tags:
        func: tracing-function
        layer: faas
        tag1: value1
        tag2: value2
      baggage:
        key: "key1"
        value: "value1"
```

建议您使用全局跟踪配置，否则您必须为您创建的每个函数逐一添加函数级跟踪配置。

## 使用 SkyWalking 作为分布式追踪解决方案

1. 通过参考 [本文档](../interact-with-dapr-output-binding/) 创建函数。

1. 然后，您可以在 SkyWalking UI 界面上观察整个函数调用的链路。

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-topology.gif)

1. 您还可以比较 Knative 运行时函数（`function-front`）在运行状态和冷启动时的响应时间。

   在冷启动时:

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-time-in-cold-start.png)

   在运行状态下:

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-time-in-running.png)
