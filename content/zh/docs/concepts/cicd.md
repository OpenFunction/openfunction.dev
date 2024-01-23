---
title: "CI/CD"
linkTitle: "CI/CD"
weight: 3500
description:
---

## 概述

以前，用户可以使用 OpenFunction 将函数或应用程序源代码构建成容器镜像，然后直接将构建的镜像部署到底层的同步/异步 Serverless 运行时，无需用户干预。

但是，OpenFunction 无法在函数或应用程序源代码更改时重新构建镜像，然后重新部署它，也无法在此镜像更改时重新部署镜像（当镜像在另一个函数中手动构建和推送时）

从 v1.0.0 开始，OpenFunction 在一个新的组件 `Revision Controller` 中添加了检测源代码或镜像更改，然后重新构建和/或重新部署新构建的镜像的能力。修订控制器能够：
- 检测 github、gitlab 或 gitee 中的源代码更改，然后在源代码更改时重新构建和重新部署新构建的镜像。
- 检测包含源代码的捆绑容器镜像（镜像）的更改，然后在捆绑镜像更改时重新构建和重新部署新构建的镜像。
- 检测函数或应用程序镜像的更改，然后在函数或应用程序镜像更改时重新部署新镜像。

## 快速开始

### 安装 `Revision Controller`

你可以在安装 [OpenFunction](https://openfunction.dev/docs/getting-started/installation/#install-openfunction) 时启用 `Revision Controller`，只需在 helm 命令中添加以下标志即可。

```shell
--set revisionController.enable=true
```

你也可以在 `OpenFunction` 安装后启用 `Revision Controller`：

```shell
kubectl apply -f https://raw.githubusercontent.com/OpenFunction/revision-controller/release-1.0/deploy/bundle.yaml
```

> `Revision Controller` 默认将安装到 `openfunction` 命名空间。如果你想将其安装到另一个命名空间，你可以下载 `bundle.yaml` 并手动更改命名空间。

### 检测源代码或镜像更改

要检测源代码或镜像更改，你需要将修订控制器开关和参数添加到函数的注解中，如下所示。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  annotations:
    openfunction.io/revision-controller: enable
    openfunction.io/revision-controller-params: |
      type: source
      repo-type: github
      polling-interval: 1m
  name: function-http-java
  namespace: default
spec:
  build:
    ...
  serving:
    ...
```

#### 注解

| 键                                            | 描述                                                                                    |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **openfunction.io/revision-controller**        | 是否启用修订控制器来检测此函数的源代码或镜像更改，可以设置为 `enable` 或 `disable`。 |
| **openfunction.io/revision-controller-params** | 修订控制器的参数。                                                            |

#### 参数

| 名称                  | 描述                                                                                                      |
| --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **type**              | 要检测的更改类型，包括 `source`、`source-image` 和 `image`。                                          |
| **polling-interval**  | 轮询镜像摘要或源代码头的间隔。                                                          |
| **repo-type**         | git 仓库的类型，包括 `github`、`gitlab` 和 `gitee`。默认为 `github`。 |
| **base-url**          | gitlab 服务器的基本 url。                                                                               |
| **auth-type**         | gitlab 服务器的认证类型。                                                                              |
| **project-id**        | gitlab 仓库的项目 id。                                                                                 |
| **insecure-registry** | 如果镜像仓库不安全，你应该将此设置为 true。                                                     |
