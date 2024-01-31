---
title: "路由"
linkTitle: "路由"
weight: 3430
description:
---

## 什么是 `Route`？

`Route` 是 `Function` 定义的一部分。`Route` 定义了来自 `Gateway` 监听器的流量如何路由到函数。

`Route` 在 `GatewayRef` 中指定了它将附加到的 `Gateway`，使其能够从 `Gateway` 接收流量。

一旦创建了同步 `Function`，函数控制器将：

- 查找 `openfunction` 命名空间中名为 `openfunction` 的 `Gateway`，然后如果函数中没有定义 `route.gatewayRef`，则附加到此 `Gateway`。
- 如果函数中没有定义 `route.hostnames`，则根据 `Gateway.spec.hostTemplate` 自动生成 `route.hostnames`。
- 如果函数中没有定义 `route.rules`，则根据路径 `/` 自动生成 `route.rules`。
- 根据 `Route` 创建一个 `HTTPRoute` 自定义资源。`BackendRefs` 将自动链接到相应的 Knative 服务修订版，并将 `HTTPRouteLabelKey` 标签添加到此 `HTTPRoute`。
- 创建服务 `{{.Name}}.{{.Namespace}}.svc.cluster.local`，此服务定义了从集群内部访问函数的入口。
- 如果 `route.gatewayRef` 引用的 `Gateway` 发生变化，将更新 `HTTPRoute`。

部署同步 `Function` 后，你将能够在 `Function` 的状态字段中找到 `Function` 地址和 `Route` 状态，例如：

```yaml
status:
  addresses:
    - type: External
      value: http://function-sample-serving-only.default.ofn.io/
    - type: Internal
      value: http://function-sample-serving-only.default.svc.cluster.local/
  build:
    resourceHash: "14903236521345556383"
    state: Skipped
  route:
    conditions:
      - message: Valid HTTPRoute
        reason: Valid
        status: "True"
        type: Accepted
    hosts:
      - function-sample-serving-only.default.ofn.io
      - function-sample-serving-only.default.svc.cluster.local
    paths:
      - type: PathPrefix
        value: /
  serving:
    lastSuccessfulResourceRef: serving-znk54
    resourceHash: "10715302888241374768"
    resourceRef: serving-znk54
    service: serving-znk54-ksvc-nbg6f
    state: Running
```

{{% alert title="注意" color="success" %}}

`Funtion.status` 中类型为 `Internal` 的地址提供了从集群内部访问函数的默认方法。
这个内部地址不受 `route.gatewayRef` 引用的 `Gateway` 的影响，它适合用作 `EventSource` 的 `sink.url`。

`Funtion.status` 中类型为 `External` 的地址提供了从集群外部访问函数的方法（你可以选择配置 Magic DNS 或真实 DNS，请参考 [通过外部地址访问函数](../function-entrypoints/#通过外部地址访问函数) 获取更多详情）。
这个外部地址是基于 `route.gatewayRef`、`router.hostnames` 和 `route.rules` 生成的。路由模式只对这个外部地址生效，下面的文档将解释它是如何工作的。

有关如何访问函数的更多信息，请参考 [函数入口点](../function-entrypoints/)。

{{% /alert %}}

## 基于主机的路由

`基于主机` 是默认的路由模式。当 `route.hostnames` 未定义时，
将根据 `gateway.spec.hostTemplate` 自动生成 `route.hostnames`。
如果 `route.rules` 未定义，将根据路径 `/` 自动生成 `route.rules`。

```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    triggers:
      http:
        route:
          gatewayRef:
            name: openfunction
            namespace: openfunction
EOF
```

> 如果你正在使用默认的 [OpenFunction Gateway](../gateway#默认的-openfunction-网关)，函数的外部地址将如下：

```shell
http://function-sample.default.ofn.io/
```

## 基于路径的路由

如果你在函数中定义了 `route.hostnames`，将根据 `gateway.spec.pathTemplate` 自动生成 `route.rules`。

```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    triggers:
      http:
        route:
          gatewayRef:
            name: openfunction
            namespace: openfunction
           hostnames:
           - "sample.ofn.io"
EOF
```

> 如果你正在使用默认的 [OpenFunction Gateway](../gateway#默认的-openfunction-网关)，函数的外部地址将如下：

```shell
http://sample.default.ofn.io/default/function-sample/
```

## 基于主机和路径的路由

你可以同时定义主机名和路径，以自定义流量应如何路由到你的函数。

{{% alert title="注意" color="success" %}}

在这种模式下，你需要自己解决 HTTPRoutes 之间可能存在的冲突。

{{% /alert %}}
```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    triggers:
      http:
        route:
          gatewayRef:
            name: openfunction
            namespace: openfunction
          rules:
            - matches:
                - path:
                    type: PathPrefix
                    value: /v2/foo
          hostnames:
          - "sample.ofn.io"
EOF
```

> 如果你正在使用默认的 [OpenFunction Gateway](../gateway#默认的-openfunction-网关)，函数的外部地址将如下：

```shell
http://sample.default.ofn.io/v2/foo/
```
