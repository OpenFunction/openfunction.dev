---
title: "函数构建"
linkTitle: "函数构建"
weight: 3110
description:
---
目前，OpenFunction支持使用[Cloud Native Buildpacks](https://buildpacks.io/)构建函数镜像，无需创建`Dockerfile`。

同时，您也可以使用OpenFunction通过`Dockerfile`构建[无服务器应用](../serverless_apps/#build-and-run-a-serverless-application-with-a-dockerfile)。

## 通过定义构建部分来构建函数

您可以从git仓库的源代码或本地存储的源代码构建函数或应用。

### 从git仓库的源代码构建函数

您可以通过在`Function`定义中添加一个构建部分来构建函数镜像。如果还定义了服务部分，那么构建完成后将立即启动函数。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: logs-async-handler
spec:
  version: "v2.0.0"
  image: openfunctiondev/logs-async-handler:v1
  imageCredentials:
    name: push-secret
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "LogsHandler"
      FUNC_CLEAR_SOURCE: "true"
      ## 自定义函数框架版本，目前仅对functions-framework-go有效
      ## 通常情况下，您不需要这样做，因为构建器会自带最新的函数框架
      # FUNC_FRAMEWORK_VERSION: "v0.4.0"
      ## 使用FUNC_GOPROXY设置goproxy
      # FUNC_GOPROXY: "https://goproxy.cn"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/async/logs-handler-function/"
      revision: "main"
```

> 要将函数镜像推送到容器仓库，您需要创建一个包含仓库凭证的秘密，并将该秘密添加到`imageCredentials`。
> 您可以参考[先决条件](../../getting-started/quickstarts/prerequisites)以获取更多信息。

### 从本地源代码构建函数

要从本地源代码构建函数或应用，您需要将本地源代码打包成容器镜像，并将此镜像推送到容器仓库。

假设您的源代码在`samples`目录中，您可以使用以下`Dockerfile`来构建源代码包镜像。

```shell
FROM scratch
WORKDIR /
COPY samples samples/
```

然后，您可以像这样构建源代码包镜像：

```shell
docker build -t <your registry name>/sample-source-code:latest -f </path/to/the/dockerfile> .
docker push <your registry name>/sample-source-code:latest
```

建议使用空镜像scratch作为构建源代码包镜像的基础镜像，非空基础镜像可能会导致源代码复制失败。  
与为git仓库方法定义spec.build.srcRepo.url字段不同，您需要定义spec.build.srcRepo.bundleContainer.image字段。

```yaml
apiVersion: core.openfunction.io/v1beta2
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

sourceSubPath是源代码包镜像中源代码的绝对路径。

## 使用pack CLI构建函数

通常，直接从本地源代码构建函数镜像是必要的，特别是出于调试目的或用于离线环境。您可以使用pack CLI来实现这一点。

Pack是由Cloud Native Buildpacks项目维护的一个工具，用于支持使用buildpacks。它启用了以下功能：

- 使用buildpacks构建应用。
- 重新基准使用buildpacks创建的应用镜像。
- 创建生态系统内使用的各种组件。

按照[这里](https://buildpacks.io/docs/tools/pack/)的说明安装`pack` CLI工具。
您可以在[这里](https://buildpacks.io/docs/tools/pack/cli/pack/)找到更多关于如何使用pack CLI的详细信息。

要从本地源代码构建OpenFunction函数镜像，您可以按照以下步骤操作：

### 下载函数示例

```shell
git clone https://github.com/OpenFunction/samples.git
cd samples/functions/knative/hello-world-go
```

使用pack CLI构建函数镜像
```shell
pack build func-helloworld-go --builder openfunction/builder-go:v2.4.0-1.17 --env FUNC_NAME="HelloWorld"  --env FUNC_CLEAR_SOURCE=true
```

本地启动函数镜像
```shell
docker run --rm --env="FUNC_CONTEXT={\"name\":\"HelloWorld\",\"version\":\"v1.0.0\",\"port\":\"8080\",\"runtime\":\"Knative\"}" --env="CONTEXT_MODE=self-host" --name func-helloworld-go -p 8080:8080 func-helloworld-go
```

访问函数
```shell
curl http://localhost:8080
```

输出示例：
```shell
hello, world!
```

## OpenFunction构建器

要使用[Cloud Native Buildpacks](https://buildpacks.io/)构建函数镜像，需要一个构建器镜像。

在这里，您可以找到OpenFunction社区维护的流行语言的构建器：

|           | 构建器                                                                                                                           |
|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| Go        | openfunction/builder-go:v2.4.0 (openfunction/builder-go:latest)                                                                    |
| Nodejs    | openfunction/builder-node:v2-16.15 (openfunction/builder-node:latest)                                                              |
| Java      | openfunction/builder-java:v2-11, openfunction/builder-java:v2-16, openfunction/builder-java:v2-17, openfunction/builder-java:v2-18 |
| Python    | openfunction/gcp-builder:v1                                                                                                        |
| DotNet    | openfunction/gcp-builder:v1                                                                                                        |