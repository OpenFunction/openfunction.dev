---
title: "简介"
linkTitle: "简介"
weight: 3410
description:
---

## 概述

从 v0.5.0 开始，OpenFunction 使用 Kubernetes Ingress 为同步函数提供统一的入口点，并且必须安装 nginx ingress 控制器。

随着 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 的成熟，我们决定在 OpenFunction v0.7.0 中实现基于 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 的 OpenFunction Gateway，以替代之前基于 ingress 的域方法。

>你可以在[这里](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/202207-openfunction-gateway.md)找到 OpenFunction Gateway 的提案

OpenFunction Gateway 提供了一个更强大、更灵活的函数网关，包括以下特性：

- 使用户能够以更简单、厂商中立的方式切换到任何支持 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 的 [网关实现](https://gateway-api.sigs.k8s.io/implementations/)，如 Contour、Istio、Apache APISIX、Envoy Gateway（未来）等。

- 用户可以选择安装默认的网关实现（Contour），然后定义一个新的 `gateway.networking.k8s.io`，或者使用他们环境中的任何现有网关实现，然后引用现有的 `gateway.networking.k8s.io`。

- 允许用户自定义他们自己的函数访问模式，如 `hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"` 用于基于主机的访问。

- 允许用户自定义他们自己的函数访问模式，如 `pathTemplate: "{{.Namespace}}/{{.Name}}"` 用于基于路径的访问。

- 允许用户在函数定义中自定义每个函数的路由规则（基于主机、基于路径或两者都有），如果没有定义自定义的路由规则，将为每个函数提供默认的路由规则。

- 直接将流量发送到 Knative 服务修订版，而不再通过 Knative 自己的网关。从 OpenFunction 0.7.0 开始，你只需要 OpenFunction Gateway 就可以访问 OpenFunction 同步函数，如果你不需要直接访问 Knative 服务，你可以忽略 Knative 的域配置错误。

- 在函数修订版之间分割流量（未来）

下图说明了客户端流量如何通过 OpenFunction Gateway，然后直接到达函数：

<div align=center><img width="100%" height="100%" src=/images/docs/en/concepts/gateway/gateway.png/></div>