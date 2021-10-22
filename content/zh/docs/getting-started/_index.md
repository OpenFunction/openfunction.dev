---
title: "入门"
linkTitle: "入门"
weight: 5
description: >
    这里将指引您快速入门 OpenFunction，包括安装及一些案例的演示
---

## 概述

**OpenFunction** 是一个云原生、开源的 FaaS（函数即服务）框架，旨在让开发人员专注于他们的开发意图，而不必关心底层运行环境和基础设施。用户只需提交一段代码，就可以生成事件驱动的、动态伸缩的 Serverless 工作负载。

OpenFunction的特点：

- 云原生，开源
- 自动构建代码为 OCI 标准镜像
- 自动部署具有动态伸缩能力的应用程序
- 提供事件框架，使函数具备事件驱动能力
- 提供函数版本控制和入口流量管理功能

OpenFunction有两个主要框架：

- 由 CustomResourceDefinitions（CRD）[Function]({{<ref "../concepts/function">}})、[Builder]({{<ref "../concepts/builder">}})、[Serving]({{<ref "../concepts/serving">}}) 提供支持的无服务计算（Serverless）框架
- 由 CustomResourceDefinitions（CRD）[EventSource]({{<ref "../concepts/eventsource">}})、[EventBus (ClusterEventBus)]({{<ref "../concepts/eventbus">}})、[Trigger]({{<ref "../concepts/trigger">}}) 提供支持的事件管理框架

## 兼容性

### Kubernetes 兼容性矩阵

我们对下列版本进行了测试验证，以确保他们可以按照所示的对应关系正常运行。但请注意，使用其他版本也可能正常运行！

| OpenFunction                                                 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 |
| ------------------------------------------------------------ | --------------- | --------------- | --------------- | --------------- |
| [`release-0.3`](https://github.com/OpenFunction/OpenFunction/tree/v0.3.0) | &radic;         | &radic;         | &radic;         | &radic;         |
| [`release-0.4`](https://github.com/OpenFunction/OpenFunction/tree/v0.4.0) | &times;         | &radic;         | &radic;         | &radic;         |
| [`HEAD`](https://github.com/OpenFunction/OpenFunction/tree/main) | &times;         | &radic;         | &radic;         | &radic;         |


## 前提准备

当前版本的 OpenFunction 要求你有一个版本为 `等于或高于 1.18.6` 的 Kubernetes 集群。

此外，你需要为 OpenFunction 的 `Builder` 和 `Serving` 部署一些依赖项。

你可以使用以下命令来安装 OpenFunction 的 `Builder` 和 `Serving`：

```shell
sh hack/deploy.sh --all
```

这个命令将安装 `Builder` 和 `Serving` 的所有依赖项到你的集群中。

你也可以用以下参数来定制安装的依赖软件。

| 参数                               | 用途说明                                                    |
| ---------------------------------- | ----------------------------------------------------------- |
| -\-all                              | 安装所有 `Builder` 和 `Serving` 的依赖软件                  |
| -\-with-shipwright                  | 安装 ShipWright（Builder 的依赖软件）                       |
| -\-with-knative                     | 安装 Knative（Serving 的依赖软件）                          |
| -\-with-openFuncAsync               | 安装 OpenFuncAsync（Serving 的依赖软件，包含 KEDA 与 Dapr） |
| -\-poor-network                     | 用于当你正使用无法有效访问 GitHub/Googleapis 的网络时       |

## 安装

你可以使用以下命令安装 OpenFunction：

- 安装最新稳定版本

```shell
kubectl create -f https://github.com/OpenFunction/OpenFunction/releases/download/v0.4.0/bundle.yaml
```

- 安装最新开发版本

```shell
kubectl create -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/main/config/bundle.yaml
```

> 注意：当使用非默认的命名空间时，确保命名空间中的 ClusterRoleBinding 是匹配的。

### 案例：运行一个简单函数

如果你已经安装了 OpenFunction，推荐参考 [OpenFunction samples](https://github.com/OpenFunction/samples) 运行一个函数案例。

## 卸载 

- 卸载最新的稳定版本

```shell
kubectl delete -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/release-0.4/config/bundle.yaml
```

- 卸载最新的开发版本

```shell
kubectl delete -f https://raw.githubusercontent.com/OpenFunction/OpenFunction/main/config/bundle.yaml
```
