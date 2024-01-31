---
title: "简介"
linkTitle: "简介"
weight: 1000
description: 
---
## 概述

OpenFunction 是一个云原生的开源 FaaS（函数即服务）平台，旨在让您专注于业务逻辑，而不必维护底层的运行环境和基础设施。您只需要以函数的形式提交与业务相关的源代码，就可以生成事件驱动和动态伸缩的 Serverless 工作负载。

<div align=center><img width="70%" height="70%" src=/images/docs/en/introduction/what-is-openfunction/function-lifecycle.svg/></div>


## 架构和设计

<div align=center><img width="80%" height="80%" src=/images/docs/en/introduction/what-is-openfunction/openfunction-architecture.svg/></div>

## 核心特性

- 云无关、和云厂商的 BaaS 服务解耦

- 可插拔架构、支持多种函数运行时

- 支持同步和异步函数

- 独有的异步函数支持，直接从事件源消费事件

- 支持从函数源代码直接生成符合 OCI 标准的镜像

- 实现从 0 到 N 的弹性自动伸缩

- 基于事件源的特定指标实现领先的异步函数自动伸缩

- 引入 [Dapr](https://dapr.io/) 以简化同步和异步函数和 BaaS 服务的集成

- [K8s Gateway API](https://gateway-api.sigs.k8s.io/) 提供了领先的函数入口和流量管理能力

- 灵活易用的事件管理框架

## License

OpenFunction 采用 Apache License 2.0 版本授权。更多信息，请参见 [LICENSE](https://github.com/OpenFunction/OpenFunction/blob/main/LICENSE).

