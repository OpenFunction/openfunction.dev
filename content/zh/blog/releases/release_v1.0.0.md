---
title: "OpenFunction v1.0.0 发布：集成 WasmEdge，支持 Wasm 函数和更完整的 CI/CD"
linkTitle: "Release v1.0.0"
date: 2023-03-11
weight: 92
---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) 是一个开源的云原生 FaaS（Function as a Service，函数即服务）平台，旨在帮助开发者专注于业务逻辑的研发。今天，我们非常高兴地宣布 OpenFunction 迎来了一次重要的更新，即 v1.0.0 版本的发布！

本次更新中，我们继续致力于为开发者们提供更加灵活和强大的工具，并在此基础上加入了一些新的功能点。其中，该版本集成了 WasmEdge 以支持 Wasm 函数；我们还对 OpenFunction 的 CI/CD 功能进行了增强，提供了相对完整的端到端的 CI/CD 功能；除此之外，我们还在这个版本中新增了从本地代码直接构建函数或应用的镜像的功能，让开发者可以更加便捷地进行代码发布和部署。

与此同时，我们也在大力优化 OpenFunction 的性能和代码质量，使其更加稳定高效。

以下是本次版本更新的主要内容：

## 集成 WasmEdge，支持 Wasm 函数

WasmEdge 是一个轻量级、高性能和可扩展的 WebAssembly 运行时，适用于云原生、边缘和去中心化应用程序。它为 Serverless 应用程序、嵌入式功能、微服务、智能合约和物联网设备提供支持。

OpenFunction 现在支持构建和运行以 WasmEdge 为运行时的 Wasm 函数或应用。WasmEdge 成为 Docker、Containerd 和 CRI-O 之外的容器运行时的新选择。

### Wasm 函数示例
```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta1
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
  port: 8080
  route:
    rules:
      - matches:
          - path:
              type: PathPrefix
              value: /echo
  serving:
    runtime: knative
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
EOF
```

借助 WasmEdge 引擎，开发者可以使用多种支持 Wasm 的语言和开发框架来编写及运行函数。

> 如何构建和运行 Wasm functions，请参考官方文档 [Wasm Functions](https://openfunction.dev/docs/concepts/wasm_functions/)。

## 更完整的 CI/CD

之前，用户可以使用 OpenFunction 将函数或应用程序源代码构建为容器镜像，然后直接将构建的镜像部署到底层的同步或异步 Serverless 运行时，而无需用户干预。

但是，OpenFunction 不能在函数或应用程序源代码发生更改时重新构建镜像并重新部署它，也不能在镜像更改时重新部署它（例如手动构建和推送镜像或在其他函数中构建镜像）。

从 v1.0.0 开始，OpenFunction 在新的组件 `Revision Controller` 中增加了检测源代码或镜像变更的能力，并且可以在检测到变更后触发镜像重新构建或重新部署新的镜像。`Revision Controller` 的能力包括：

- 检测 GitHub、GitLab 或 Gitee 中的源代码变更，然后在源代码变更时重新构建并重新部署新的镜像。
- 检测包含源代码的镜像（Bundle Container Image）的变更，然后在该镜像变更时重新构建和重新部署新的镜像。
- 检测函数或应用程序镜像变更，然后在函数或应用程序镜像变更时重新部署新的镜像。

更好的 CI/CD 功能确保了代码能在不同的环境中高效运行，使用者可以在开发和部署过程中更好的控制版本和代码质量，同时也为使用者提供了更好的使用体验。

> 此处请参考官方文档 [CI/CD](https://openfunction.dev/docs/concepts/cicd/)。

## 从本地源代码构建函数

目前，OpenFunction v1.0.0 支持根据本地的源代码构建函数或应用。只需要将本地源代码打包到容器镜像中，并将此镜像推送到容器镜像仓库即可完成构建。以下为操作方法。

假设你的源代码在 `samples` 目录中，你可以根据以下 `Dockerfile` 来构建包含源代码的镜像。

```shell
FROM scratch
WORKDIR /
COPY samples samples/
```

然后如下操作：

```shell
docker build -t <your registry name>/sample-source-code:latest -f </path/to/the/dockerfile> .
docker push <your registry name>/sample-source-code:latest
```

> 推荐使用空镜像 `scratch` 作为基础镜像构建包含源代码的镜像，非空基础镜像可能会导致源代码拷贝失败。

另外还需要定义字段 `spec.build.srcRepo.bundleContainer.image`。

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: logs-async-handler
spec:
  build:
    srcRepo:
      bundleContainer:
        image: openfunctiondev/sample-source-code:latest
      sourceSubPath: "/samples/functions/async/logs-handler-function/"
```

> `sourceSubPath` 是包含源代码的镜像中源代码的绝对路径。

## 其他的改进和优化

除了上述的主要变化，该版本还有以下更改和增强：

- OpenFunction
  - 核心 API `v1alpha2` 已弃用并删除
  - 将 sha256 添加到服务镜像
  - 将构建源信息添加到函数状态
  - 将 Shipwright 升级到 v0.11.0
  - 将 Knative 升级到 v0.32.0
  - 将 Dapr 升级到 v1.8.3
  - 将 Go 升级到 v1.18
- functions-framework-java 发布 V1.0.0
  - 在一个 pod 中支持多个函数
  - 支持自动发布
- Builder
  - 在一个 pod 中支持多个函数
  - 将默认的 Java 框架版本更新为 1.0.0
- revision-controller 发布 v1.0.0（功能见上文）


以上就是 OpenFunction v1.0.0 的主要功能变化，在此十分感谢各位贡献者的参与和贡献。如果您正在寻找一款高效、灵活的云原生函数开发平台，那么 OpenFunction v1.0.0 将是您的绝佳选择。

了解更多关于 OpenFunction 和本次版本更新的信息，欢迎访问我们的官方网站和 Github 页面。

- [官网](https://openfunction.dev/)：https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.0.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.0.0