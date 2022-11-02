---
title: "OpenFunction v0.8.0 发布：通过 Dapr Proxy 加速函数启动"
linkTitle: "Release v0.8.0"
date: 2022-11-02
weight: 93
---

相较于其他函数计算项目，OpenFunction 有很多独特的功能，其中之一便是通过 Dapr 与各种后端服务（BaaS）无缝集成。目前 OpenFunction 已经支持了 Dapr 的 pub/sub 和 bindings 构建模块，未来还会支持更多功能。截止到 v0.7.0，OpenFunction 与 BaaS 的集成还不算特别丝滑，需要在每个函数实例的 Pod 中注入一个 Dapr Sidecar 容器，这就会导致一个问题：**整个函数实例的启动时间会受到 Dapr Sidecar 容器启动时间的影响**，甚至 Dapr Sidecar 容器可能会比函数容器本身消耗的资源更多。

为了解决这个问题，OpenFunction 发布了 v0.8.0，引入了 Dapr Standalone 模式。

## Dapr Standalone 模式

在新的 Dapr Standalone 模式中，无需为每个函数实例启动一个独立的 Dapr Sidecar 容器，而是为每个函数创建一个 Dapr Proxy 服务，这个函数的所有实例都会共享这个 Dapr Proxy 服务，大大减少了函数的启动时间。

![](https://pek3b.qingstor.com/kubesphere-community/images/202211011453291.png)

## 如何选择 Dapr 服务模式

当然，除了新增的 Dapr Standalone 模式之外，您还可以继续选择使用 Dapr Sidecar 模式，建议您根据自己的实际业务来选择合适的 Dapr 服务模式。默认推荐使用 Dapr Standalone 模式，如果您的函数实例不需要频繁伸缩，或者由于一些其他原因无法使用 Dapr Standalone 模式，则可以使用 Dpar Sidecar 模式。

您可以通过在自定义资源 Function 中添加 `spec.serving.annotations` 来控制使用何种 Dapr 模式与 BaaS 集成，总共有两个 annotations：

+ `openfunction.io/enable-dapr`：可以设置为 `true` 或者 `false`；
+ `openfunction.io/dapr-service-mode`：可以设置为 `standalone` 或者 `sidecar`。

当 `openfunction.io/enable-dapr` 设置为 `true` 时，可以调整 `openfunction.io/dapr-service-mode` 的值来选择不同的 Dapr 服务模式。

当 `openfunction.io/enable-dapr` 设置为 `false` 时，`openfunction.io/dapr-service-mode` 的值会被忽略，Dapr Standalone 模式与 Dapr Sidecar 模式都不会启用。

这两个 annotations 的默认值如下：

+ 如果 Function 中没有添加 `spec.serving.annotations` 配置块，并且 Function 中包含 `spec.serving.inputs` 和 `spec.serving.outputs` 配置块，`openfunction.io/enable-dapr` 会被自动设置为 `true`；否则 `openfunction.io/enable-dapr` 就会被设置为 `false`。
+ `openfunction.io/dapr-service-mode` 默认值是 `standalone`。

示例：

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: cron-input-kafka-output
spec:
  ...
  serving:
    annotations:
      openfunction.io/enable-dapr: "true"
      openfunction.io/dapr-service-mode: "standalone"
    template:
      containers:
        - name: function # DO NOT change this
          imagePullPolicy: IfNotPresent 
    runtime: "async"
    inputs:
      - name: cron
        component: cron
    outputs:
      - name: sample
        component: kafka-server
        operation: "create"
    ...
```

关于 v0.8.0 的更多发版细节可以参考 OpenFunction v0.8.0 的 [Release Notes](https://github.com/OpenFunction/OpenFunction/releases/tag/v0.8.0)。