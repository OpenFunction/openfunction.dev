---
title: "OpenFunction 0.7.0 发布: OpenFunction Gateway、多语言及 Helm 安装支持"
linkTitle: "Release v0.7.0"
date: 2022-08-19
weight: 94

---
[OpenFunction](https://github.com/OpenFunction/OpenFunction) 是一个开源的云原生 FaaS（Function as a
Service，函数即服务）平台，旨在帮助开发者专注于业务逻辑的研发。在过去的几个月里，OpenFunction 社区一直在努力工作，为 OpenFunction 0.7.0 版本的发布做准备。今天，我们非常高兴地宣布
OpenFunction 0.7.0 已经正式发布了！感谢社区各位小伙伴的贡献和反馈！

OpenFunction 0.7.0 为您带来了许多新功能，包括新增 OpenFunction Gateway 作为同步函数入口、 新增 Java 和 NodeJS 同步函数和异步函数支持、新增 Helm 安装方式。
同时， 我们对 OpenFunction 依赖的组件都进行了版本升级。

### OpenFunction Gateway

OpenFunction 从 0.5.0 开始采用 Kubernetes Ingress 来提供同步函数的统一入口，并且必须安装一个 nginx-ingress-controller。

在 OpenFunction 0.7.0 中，我们基于 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 实现了 OpenFunction Gateway 替代之前基于
Kubernetes Ingress 的 Domain 来访问同步函数的方法。

OpenFunction Gateway 提供了更强大、更灵活的函数网关，包含以下特性：

- 可以选择任意支持 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 的 [Gateway 实现](https://gteaway-api.sigs.k8s.io/implementations/)，如 Contour, Istio, Apache APISIX, Envoy Gateway 等。
- 可以选择安装默认的 Gateway 实现（Contour）, 此时 OpenFunction 将自动创建 Kubernetes Gateway。OpenFunction 也可以使用您环境中现有的 Kubernetes Gateway，只需要您在 OpenFunction Gateway 中引用它即可。
- 可以自定义访问函数的模式，如基于主机的路由模式和基于路径的路由模式，在您没有定义函数路由时 OpenFunction 默认提供基于主机的路由模式来访问函数。
- 可以在函数路由部分自定义流量应该如何到达函数，OpenFunction 基于 [Gateway API HTTPRoute](https://gateway-api.sigs.k8s.io/api-types/httproute/) 为您提供了强大的函数路由功能。
- 可以通过函数外部地址在集群外部访问函数，只需要在OpenFunction Gateway 中配置好集群外部可以访问的域名即可（同时支持 Magic DNS 和 Real DNS）。
- 现在 OpenFunction 将流量直接转发到 Knative Revision 而不再经过 Knative 的 Gateway。 如果不需要直接访问 Knative 服务, 您可以忽略 Knative Domain 相关的错误。

将来，OpenFunction 将支持在函数的不同版本之间进行流量分发。

### 多语言支持
OpenFunction 社区一直在努力完善多语言的支持：
- Go

  [functions-framework-go](https://github.com/OpenFunction/functions-framework-go) 发布了 0.4.0，支持在一个函数中定义多个子函数，并且可以通过不同的 Path 和 Method 进行分别调用。

- Java
  
  [functions-framework-java](https://github.com/OpenFunction/functions-framework-java) 现在支持同步函数和异步函数。

- NodeJS

  [functions-framework-nodejs](https://github.com/OpenFunction/functions-framework-nodejs) 发布了 0.5.0， 支持同步函数和异步函数，并且支持同步函数触发异步函数。

  我们将在近期发布 [functions-framework-nodejs](https://github.com/OpenFunction/functions-framework-nodejs) 0.6.0，为您带来更多功能。

OpenFunction 将会在后续版本支持更多语言如 Python、Dotnet 等。

### Helm 安装 OpenFunction 及所有依赖组件

_原来基于 CLI 安装的方法已弃用。_

现在 OpenFunction 支持通过 Helm 安装 OpenFunction 及所有依赖的组件，相比原来通过 CLI 安装的方式更加云原生， 并且解决了部分用户访问 Google Container Registry（gcr.io）镜像受限的问题， 并且将长期维护。

**TL;DR**
```
helm repo add openfunction https://openfunction.github.io/charts/
helm repo update
helm install openfunction openfunction/openfunction -n openfunction --create-namespace
```

### 依赖组件升级

| Components             | OpenFunction 0.6.0 | OpenFunction 0.7.0 |
| ---------------------- |--------------------|--------------------|
| Knative Serving        | 1.0.1              | 1.3.2              |
| Dapr                   | 1.5.1              | 1.8.3              |
| Keda                   | 2.4.0              | 2.7.1              |
| Shipwright             | 0.6.1              | 0.10.0             |
| Tekton Pipelines       | 0.30.0             | 0.37.0             |
