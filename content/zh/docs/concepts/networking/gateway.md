---
title: "网关"
linkTitle: "网关"
weight: 3420
description:
---

## 深入理解 OpenFunction 网关

OpenFunction 网关由 [Kubernetes 网关](https://gateway-api.sigs.k8s.io/)支持，定义了用户如何访问同步函数。

每当创建一个 `OpenFunction 网关` 时，网关控制器将：

- 如果 `gatewaySpec.listeners` 中没有，将添加一个名为 `ofn-http-internal` 的默认监听器。

- 根据 `domain` 或 `clusterDomain` 生成 `gatewaySpec.listeners.[*].hostname`。

- 将 `gatewaySpec.listenters` 注入到 `OpenFunction 网关` 的 `gatewayRef` 定义的现有 `Kubernetes 网关` 中。

- 根据 `OpenFunction 网关` 的 `gatewayDef` 中的 `gatewaySpec.listenters` 字段创建一个新的 `Kubernetes 网关`。

- 创建一个名为 `gateway.openfunction.svc.cluster.local` 的服务，该服务定义了同步函数的统一入口。

部署 `OpenFunction 网关` 后，你将能够在 `OpenFunction 网关` 状态中找到 `Kubernetes 网关` 和其 `listeners` 的状态：

```yaml
status:
  conditions:
  - message: Gateway is scheduled
    reason: Scheduled
    status: "True"
    type: Scheduled
  - message: Valid Gateway
    reason: Valid
    status: "True"
    type: Ready
  listeners:
  - attachedRoutes: 0
    conditions:
    - message: Valid listener
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-internal
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
  - attachedRoutes: 0
    conditions:
    - message: Valid listener
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-external
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
```

## 默认的 OpenFunction 网关

`OpenFunction 网关` 使用 `Contour` 作为默认的 `Kubernetes 网关` 实现。
安装 OpenFunction 后，将自动创建以下 `OpenFunction 网关`：

```yaml
apiVersion: networking.openfunction.io/v1alpha1
kind: Gateway
metadata:
  name: openfunction
  namespace: openfunction
spec:
  domain: ofn.io
  clusterDomain: cluster.local
  hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"
  pathTemplate: "{{.Namespace}}/{{.Name}}"
  httpRouteLabelKey: "app.kubernetes.io/managed-by"
  gatewayRef:
    name: contour
    namespace: projectcontour
  gatewaySpec:
    listeners:
      - name: ofn-http-internal
        hostname: "*.cluster.local"
        protocol: HTTP
        port: 80
        allowedRoutes:
          namespaces:
            from: All
      - name: ofn-http-external
        hostname: "*.ofn.io"
        protocol: HTTP
        port: 80
        allowedRoutes:
          namespaces:
            from: All
```

你可以像下面这样自定义默认的 `OpenFunction 网关`：
```shell
kubectl edit gateway openfunction -n openfunction
```

## 切换到不同的 Kubernetes 网关

你可以以更简单、厂商中立的方式切换到任何支持 [Kubernetes 网关 API](https://gateway-api.sigs.k8s.io/) 的 [网关实现](https://gateway-api.sigs.k8s.io/implementations/)，如 Contour、Istio、Apache APISIX、Envoy Gateway（未来）等。

> [这里](../../../operations/networking/switch-gateway) 你可以找到更多详细信息。

## 多个 OpenFunction 网关
对于 OpenFunction，多个 `Gateway` 没有意义，我们目前只支持一个 OpenFunction `Gateway`。