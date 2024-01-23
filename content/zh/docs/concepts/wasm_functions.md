---
title: "Wasm 函数"
linkTitle: "Wasm 函数"
weight: 3310
description:
---

`WasmEdge` 是一个轻量级、高性能、可扩展的 WebAssembly 运行时，用于云原生、边缘和去中心化应用。它支持无服务器应用、嵌入式函数、微服务、智能合约和物联网设备。

OpenFunction 现在支持使用 `WasmEdge` 作为工作负载运行时来构建和运行 wasm 函数。

> 你可以在[这里](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/20230223-wasmedge-integration.md)找到 WasmEdge 集成提案

## Wasm 容器镜像

包含 wasm 二进制文件的 wasm 镜像是一个没有操作系统层的特殊容器镜像。应该在这个 wasm 容器镜像中添加一个特殊的注解 `module.wasm.image/variant: compat-smart`，以便像 WasmEdge 这样的 wasm 运行时能够识别它。这在 OpenFunction 中是自动处理的，用户只需要将 `workloadRuntime` 指定为 `wasmedge`。

### Wasm 容器镜像的构建阶段

如果 `function.spec.workloadRuntime` 设置为 `wasmedge`，或者函数的注解包含 `module.wasm.image/variant: compat-smart`，`function.spec.build.shipwright.strategy` 将根据名为 `wasmedge` 的 `ClusterBuildStrategy` 自动生成，以便构建带有 `module.wasm.image/variant: compat-smart` 注解的 wasm 容器镜像。

### Wasm 容器镜像的服务阶段

当 `function.spec.workloadRuntime` 设置为 `wasmedge`，或者函数的注解包含 `module.wasm.image/variant: compat-smart` 时：
- 如果 `function.spec.serving.annotations` 不包含 `module.wasm.image/variant`，`module.wasm.image/variant: compat-smart` 将自动添加到 `function.spec.serving.annotations`。
- 如果 `function.spec.serving.template.runtimeClassName` 没有设置，这个 `runtimeClassName` 将自动设置为默认的 `openfunction-crun`

> 如果你的 kubernetes 集群在像 `Azure` 这样的公共云中，你可以手动设置 `spec.serving.template.runtimeClassName` 来覆盖默认的 `runtimeClassName`。

## 构建和运行 wasm 函数

> 要在 kubernetes 集群中设置 `WasmEdge` 工作负载运行时并将镜像推送到容器镜像仓库，
> 请参考[先决条件](../../getting-started/Quickstarts/prerequisites)部分以获取更多信息。

> 你可以在[这里](https://github.com/OpenFunction/samples/tree/main/functions/knative/wasmedge/http-server)找到关于这个示例函数的更多信息。

1. 创建一个 wasm 函数

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: wasmedge-http-server
spec:
  workloadRuntime: wasmedge
  image: openfunctiondev/wasmedge_http_server:0.1.0
  imageCredentials:
    name: push-secret
  build:
    dockerfile: Dockerfile
    srcRepo:
      revision: main
      sourceSubPath: functions/knative/wasmedge/http-server
      url: https://github.com/OpenFunction/samples
  serving:
    scaleOptions:
      minReplicas: 0
    template:
      containers:
        - command:
            - /wasmedge_hyper_server.wasm
          imagePullPolicy: IfNotPresent
          livenessProbe:
            initialDelaySeconds: 3
            periodSeconds: 30
            tcpSocket:
              port: 8080
          name: function
    triggers:
      http:
        port: 8080
        route:
          rules:
            - matches:
                - path:
                    type: PathPrefix
                    value: /echo
EOF
```

2. 检查 wasm 函数的状态

```shell
kubectl get functions.core.openfunction.io -w
NAME                   BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                      AGE
wasmedge-http-server   Succeeded    Running        builder-4p2qq   serving-lrd8c   http://wasmedge-http-server.default.svc.cluster.local/echo   12m
```

3. 访问 wasm 函数

一旦 `BUILDSTATE` 变为 `Succeeded`，`SERVINGSTATE` 变为 `Running`，你就可以通过 `ADDRESS` 字段中的地址访问这个函数：

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://wasmedge-http-server.default.svc.cluster.local/echo  -X POST -d "WasmEdge"
WasmEdge
```