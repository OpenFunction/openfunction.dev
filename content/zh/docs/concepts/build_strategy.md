---
title: "构建策略"
linkTitle: "构建策略"
weight: 3120
description:
---

构建策略用于控制构建过程。有两种类型的策略，`ClusterBuildStrategy` 和 `BuildStrategy`。
这两种策略都定义了一组步骤，这些步骤是控制应用程序构建过程所必需的。

`ClusterBuildStrategy` 是集群范围的，而 `BuildStrategy` 是命名空间范围的。

在 OpenFunction 中有 4 种内置的 `ClusterBuildStrategy`，你可以在以下各节中找到更多详细信息。

## openfunction

`openfunction` ClusterBuildStrategy 使用 [Buildpacks](https://buildpacks.io/docs/) 来构建函数镜像，这是默认的构建策略。

以下是 `openfunction` ClusterBuildStrategy 的参数：

| 名称 | 类型 | 描述 |
| --- | --- | --- |
| RUN_IMAGE | string | 使用的运行镜像的引用 |
| CACHE_IMAGE | string | 缓存镜像是一种在不同构建之间保留缓存层的方式，当构建具有大量依赖项的函数或应用程序（如 Java 函数）时，可以提高构建性能。 |
| BASH_IMAGE | string | 策略使用的 bash 镜像。  |
| ENV_VARS  | string | 在 _构建时_ 设置的环境变量。格式为 `key1=value1,key2=value2`。 |

用户可以像这样设置这些参数：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: logs-async-handler
spec:
  build:
    shipwright:
      params:
        RUN_IMAGE: ""
        ENV_VARS: ""
```

## buildah

`buildah` ClusterBuildStrategy 使用 [buildah](https://buildah.io/) 来构建应用程序镜像。

要使用 `buildah` ClusterBuildStrategy，你可以定义一个 `Function`，如下所示：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: sample-go-app
  namespace: default
spec:
  build:
    builder: openfunction/buildah:v1.23.1
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: buildah
```

以下是 `buildah` ClusterBuildStrategy 的参数：

| 名称 | 类型 | 描述 |  默认值 |
| --- | --- | --- | --- |
| registry-search   | string | 用于搜索短名称镜像（如 `golang:latest`）的镜像仓库，用逗号分隔。 | docker.io,quay.io |
| registry-insecure | string | 不安全镜像仓库的全名。不安全的镜像仓库是没有有效 SSL 证书或只支持 HTTP 的镜像仓库。 |
| registry-block    | string | 需要阻止拉取访问的镜像仓库。 | "" |

## kaniko

`kaniko` ClusterBuildStrategy 使用 [kaniko](https://github.com/GoogleContainerTools/kaniko) 来构建应用程序镜像。

要使用 `kaniko` ClusterBuildStrategy，你可以定义一个 `Function`，如下所示：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-kaniko
  namespace: default
spec:
  build:
    builder: openfunction/kaniko-executor:v1.7.0
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: kaniko
```

## ko

`ko` ClusterBuildStrategy 使用 [ko](https://github.com/ko-build/ko) 来构建 `Go` 应用程序镜像。

要使用 `ko` ClusterBuildStrategy，你可以定义一个 `Function`，如下所示：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-ko
  namespace: default
spec:
  build:
    builder: golang:1.17
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: ko
```

以下是 `ko` ClusterBuildStrategy 的参数：

| 名称 | 类型 | 描述 |  默认值 |
| --- | --- | --- | --- |
| go-flags          | string | Value for the GOFLAGS environment variable. | "" |
| ko-version        | string | Version of ko, must be either 'latest', or a release name from https://github.com/google/ko/releases. | "" |
| package-directory | string | The directory inside the context directory containing the main package. | "." |

## 自定义策略

用户可以自定义他们自己的策略。要自定义策略，你可以参考 [这里](https://github.com/shipwright-io/build/blob/main/docs/buildstrategies.md)。