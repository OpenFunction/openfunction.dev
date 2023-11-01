---
title: "OpenFunction v1.2.0 发布：集成 KEDA http-addon 作为同步函数运行时"
linkTitle: "Release v1.2.0"
date: 2023-11-01
weight: 92
---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) 是一个开源的云原生 FaaS（Function as a Service，函数即服务）平台，旨在帮助开发者专注于业务逻辑的研发。我们非常高兴地宣布 OpenFunction 又迎来了一次重要的更新，即 v1.2.0 版本的发布！

本次更新中，我们继续致力于为开发者们提供更加灵活和强大的工具，并在此基础上加入了一些新的功能点。该版本集成了 KEDA http-addon 作为同步函数运行时；支持在启用 SkyWalking 跟踪时添加环境变量；支持记录构建时间等。此外，还升级了部分组件及修复了多项 bug。

以下是该版本更新的主要内容：

## 集成 KEDA http-addon 作为同步函数运行时

KEDA http-addon 是一个 KEDA 的附加组件，它可以根据 HTTP 流量的变化自动地调整 HTTP 服务器的规模（包括从零开始扩容和缩容到零）。

KEDA http-addon 的工作原理是，它会在 Kubernetes 集群中创建一个名为 Interceptor 的组件，用来接收所有的 HTTP 请求，并将请求转发给目标应用。同时，它会将请求队列的长度报告给一个名为 External Scaler 的组件，用来触发 KEDA 的自动扩缩容机制。这样，你的 HTTP 应用就可以根据实际的流量需求动态地调整副本数。

在 OpenFunction v1.2.0 版本中，我们集成了 KEDA http-addon 作为同步函数运行时的一种选择。这意味着，你可以使用 OpenFunction 来创建和管理基于 HTTP 的函数，并利用 KEDA http-addon 的能力来实现高效且灵活的弹性伸缩。你只需在创建 Function 资源时指定 `serving.triggers[*].http.engine` 的值为 keda ，并且在 `serving.scaleOptions` 中配置 `keda.httpScaledObject` 相关参数，就可以部署和运行你的 HTTP 函数了。

## 支持在启用 SkyWalking 跟踪时添加环境变量

SkyWalking 是一个开源的应用性能监控（APM）系统，它可以帮助你观察和分析你的应用在不同环境中的运行状况。OpenFunction 支持在部署函数时启用 SkyWalking 跟踪，以便你可以更好地理解和优化你的函数性能。

在 OpenFunction v1.2.0 版本中，我们增加了一个新的功能，即支持在启用 SkyWalking 跟踪时添加环境变量。这样，你可以在创建 Function 资源时指定一些自定义的环境变量来控制 SkyWalking 的一些配置参数。这些环境变量会被传递给函数容器，并影响 SkyWalking 的采集和上报行为。

## 当 Function、Builder 和 Serving 状态变化时支持记录事件

事件（Event）是 Kubernetes 中一种重要的资源类型，它可以记录集群中发生的一些重要或者有趣的事情。事件可以帮助用户和开发者了解集群中资源的状态变化和异常情况，并采取相应的措施。

在 OpenFunction v1.2.0 版本中，我们支持当 Function、Builder 和 Serving 状态变化时记录事件。这样，你可以通过查看事件来获取更多关于函数构建和运行过程中发生的事情的信息。例如，你可以看到函数构建开始、结束、失败等事件；函数运行时创建、更新、删除等事件。

## 其他的改进和优化

除了上述的主要变化，该版本还有以下更改和增强：

- 升级了 KEDA 到 v2.10.1 ，HPA（自动伸缩）API 版本到 v2 ，提高了稳定性和兼容性
- 支持记录构建时间，以便你可以了解函数构建的耗时情况
- 调整了 CI 流程，修复了一些小问题
- 修复了一个在 keda http-addon 运行时中的 bug ，该 bug 会导致函数无法正常运行
- 升级了 charts 中的一些组件，包括 keda ，dapr 和 contour ，以保持最新的版本和功能

以上就是 OpenFunction v1.2.0 的主要功能变化，在此十分感谢各位贡献者的参与和贡献。

了解更多关于 OpenFunction 和本次版本更新的信息，欢迎访问我们的官方网站和 Github 页面。

- [官网](https://openfunction.dev/)：https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.2.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.2.0