---
title: "函数定义"
linkTitle: "函数定义"
weight: 3100
description:
---
## 函数

`Function`是`Build`和`Serving`的控制平面，也是用户使用OpenFunction的接口。用户无需单独创建`Build`或`Serving`，因为`Function`是定义函数的`Build`和`Serving`的唯一地方。

一旦创建了函数，它将控制`Build`和`Serving`的生命周期，无需用户干预：

- 如果在函数中定义了`Build`，则一旦部署函数，将创建一个builder自定义资源来构建函数的容器镜像。

- 如果在函数中定义了`Serving`，则将创建一个serving自定义资源来控制函数的服务和自动缩放。

- `Build`和`Serving`可以一起定义，这意味着首先将构建函数镜像，然后将其用于服务。

- 可以定义`Build`而不定义`Serving`，在这种情况下，函数仅用于构建镜像。

- 可以定义`Serving`而不定义`Build`，函数将使用先前构建的函数镜像进行服务。

<div align=center><img width="80%" height="80%" src=/images/docs/en/introduction/what-is-openfunction/openfunction-architecture.svg/></div>

## 构建

OpenFunction使用[Shipwright](https://github.com/shipwright-io/build)和[Cloud Native Buildpacks](https://buildpacks.io/)将函数源代码构建成容器镜像。

一旦创建了包含`Build`规范的函数，就会创建一个`builder`自定义资源，该资源将使用Shipwright管理构建工具和策略。然后，Shipwright将使用Tekton控制构建容器镜像的过程，包括获取源代码、生成镜像工件和发布镜像。

<div align=center><img width="60%" height="60%" src=/images/docs/en/concepts/function/openfunction-build.svg/></div>

## 服务
一旦创建了包含`Serving`规范的函数，就会创建一个`Serving`自定义资源来控制函数的服务阶段。目前，OpenFunction Serving支持两种运行时：Knative同步运行时和OpenFunction异步运行时。

## 同步运行时

对于同步函数，OpenFunction目前支持使用 Knative Serving 和 KEDA http-addon 作为运行时。

## 异步运行时

OpenFunction的异步运行时是一个基于[KEDA](https://keda.sh/)和[Dapr](https://dapr.io/)实现的事件驱动运行时。异步函数可以由各种事件类型触发，如消息队列、cronjob和MQTT等。

<div align=center><img width="70%" height="70%" src=/images/docs/en/concepts/function/openfunction-serving.svg/></div>
