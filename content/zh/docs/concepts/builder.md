---
title: "Builder"
linkTitle: "Builder"
weight: 15
description: >
    OpenFunction 概念之核心框架资源，函数构建器 Builder
---

**Builder** 定义了 OpenFunction 中由源代码生成应用镜像的构建工作。

当前，OpenFunction Builder 使用 [Shipwright](#shipwright) 和 [Cloud Native Buildpacks](#cloud-native-buildpacks) 来构建应用镜像。它通过 Shipwright 控制应用镜像的构建过程，包括通过 Cloud Native Buildpacks 获取代码、生成镜像制品和发布镜像。

### 支持的构建器

- [OpenFunction Builder](https://github.com/OpenFunction/builder)
- [GoogleCloudPlatform Builder](https://github.com/GoogleCloudPlatform/buildpacks)
- [Paketo Builder](https://paketo.io/docs/)

### Shipwright

[Shipwright](https://github.com/shipwright-io/build) 是一个可扩展的镜像流水线框架，用于在 Kubernetes 上构建容器镜像。

### Cloud Native Buildpacks

[Cloud Native Buildpacks](https://buildpacks.io/) 是一个 OCI 标准的镜像构建框架，无需 Dockerfile 即可根据自定义步骤（buildpack）将源代码转化为容器镜像制品。

### 参考

Builder 是一种 [CustomResourceDefinitions（CRD）](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)。你可以访问 [Builder CRD Spec]({{< ref "../reference/function-spec#buildimpl" >}}) 了解更多信息。
