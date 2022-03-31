---
title: "OpenFunction 0.6.0 发布: FaaS 可观测性、HTTP 同步函数能力增强及更多特性"
linkTitle: "Release v0.6.0"
date: 2022-03-25
weight: 95

---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) 是一个开源的云原生 FaaS（Function as a Service，函数即服务）平台，旨在帮助开发者专注于业务逻辑的研发。在过去的几个月里，OpenFunction 社区一直在努力工作，为 OpenFunction 0.6.0 版本的发布做准备。今天，我们非常高兴地宣布 OpenFunction 0.6.0 已经正式发布了！感谢社区各位小伙伴对新功能、增强功能和错误修复的各种帮助！

OpenFunction 0.6.0 为您带来了许多值得关注的功能，包括函数插件、函数的分布式跟踪、控制自动缩放、HTTP 函数触发异步函数等。同时，异步运行时定义也被重构了。核心 API 也已经从 `v1alpha1` 升级到 `v1beta1`。

### 面向 Serverless 函数的分布式追踪

当试图了解和诊断分布式系统和微服务时，最有效的方法之一是通过追踪函数的调用链路。分布式追踪为 Serverless 函数提供了一个关于消息流动和分布式事务监控方式的整体视图。OpenFunction 团队与 [Apache SkyWalking](https://skywalking.apache.org/) 社区合作，增加了 FaaS 的可观测性，使得您可以在 SkyWalking UI 上通过图表来可视化 Serverless 函数的依赖关系并追踪函数的调用。

![openfunction-tracing-skywalking](/images/docs/en/blogs/openfunction-tracing-skywalking.jpg)

将来，OpenFunction 将在日志、指标和追踪方面为 Serverless 功能增加更多的功能。您将能够使用 Apache SkyWalking 和 OpenFunction，为您的 Serverless 工作负载建立一个开箱即用的全栈 APM（Application Performance Monitoring）。此外，OpenFunction 将支持 OpenTelemetry，帮助您利用 Jaeger 或 Zipkin 作为分布式追踪的其它选项。

### 支持 Dapr 发布/订阅和绑定（Binding）

[Dapr Binding](https://docs.dapr.io/reference/components-reference/supported-bindings/) 允许您使用来自外部系统的事件触发您的应用程序或服务，或与外部系统对接。OpenFunction 0.6.0 为其同步函数增加了 Dapr 输出绑定（Output Binding），使异步函数通过 HTTP 同步函数进行触发成为了可能。例如，由 Knative 运行时支持的同步函数现在可以与由 Dapr 输出绑定或 [Dapr Pub/Sub 中间件](https://docs.dapr.io/reference/components-reference/supported-pubsub/) 进行交互，异步函数将被同步函数发送的事件所触发。您可以通过这个 [指南](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding) 获得快速入门样例。

异步函数则引入了 [Dapr Pub/Sub](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/)，提供一个平台无关的 API 来发送和接收消息。一个典型的用例是，您可以利用同步函数来接收纯 JSON 或 [Cloud Event](https://cloudevents.io/) 格式的事件，然后将收到的事件发送到 Dapr 输出绑定或 Pub/Sub 组件，比如是一个消息队列（如 Kafka、NATS Streaming、GCP PubSub）。最后，异步函数可以从消息队列中被触发。您可以通过这个 [指南](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub)  获得快速入门样例。

![http-trigger-openfunction](/images/docs/en/blogs/http-trigger-openfunction.jpg)

### 函数的自动伸缩行为控制

OpenFunction 0.6.0 集成了 KEDA [ScaledObject](https://keda.sh/docs/2.5/concepts/scaling-deployments/#scaledobject-spec) 规范，用于定义 KEDA 应该如何扩展您的应用程序以及触发器是什么。您只需要在 OpenFunction 函数的 CRD 中定义伸缩下限和上限，而无需改变您的代码。

同时，OpenFunction 社区也在开发控制并发性和同时请求数的能力，它继承了 [Dapr](https://docs.dapr.io/operations/configuration/control-concurrency/) 和 [Knative](https://knative.dev/docs/serving/autoscaling/concurrency/) 的定义。分布式计算的一个典型用例是只允许一定数量的请求同时执行。您将能够控制多少个请求和事件将同时调用您的应用程序。这个功能将在下一个版本中得到完全支持，敬请期待！如果您对这个功能感兴趣，请查看官方代码仓库中的 [讨论](https://github.com/OpenFunction/OpenFunction/issues/165)，了解详细的背景。

### 在实践中学习

OpenFunction 的创始人霍秉杰先生在 Dapr 社区会议上介绍了 OpenFunction 0.6.0 的两个典型用例：

1. [HTTP trigger for asynchronous functions with OpenFunction and Kafka](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding)
2. [Elastic Kubernetes log alerts with OpenFunction and Kafka](https://github.com/OpenFunction/samples/tree/main/functions/knative/logs-handler-function)

您可以观看下面这两段视频，并按照实践指南进行练习。

<iframe src="//player.bilibili.com/player.html?aid=510142597&bvid=BV1zu411v7rp&cid=562833539&page=1&high_quality=1&danmaku=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 640px; height: 430px; max-width: 100%"> </iframe>

<iframe src="//player.bilibili.com/player.html?aid=767412996&bvid=BV1mr4y1i71W&cid=554331549&page=1&high_quality=1&danmaku=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 640px; height: 430px; max-width: 100%"></iframe>

您还可以从 [发布说明](https://github.com/OpenFunction/OpenFunction/releases/tag/v0.6.0) 中了解更多关于 OpenFunction 0.6.0 的信息。参照 [快速入门](https://github.com/OpenFunction/OpenFunction#-quickstart) 和 [样例](https://github.com/OpenFunction/samples) 开始使用 OpenFunction。
