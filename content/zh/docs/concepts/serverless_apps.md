---
title: "无服务器应用"
linkTitle: "无服务器应用"
weight: 3330
description:
---
除了构建和运行无服务器函数外，你还可以使用 OpenFuntion 构建和运行无服务器应用。

OpenFunction 支持两种不同的方式将源代码构建成容器镜像：
- 使用 [Cloud Native Buildpacks](https://buildpacks.io/) 在没有 `Dockerfile` 的情况下构建源代码
- 使用 [Buildah](https://buildah.io/) 或 [BuildKit](https://github.com/moby/buildkit) 在有 `Dockerfile` 的情况下构建源代码

> 要将镜像推送到容器镜像仓库，你需要创建一个包含镜像仓库凭据的 secret，并将 secret 添加到 `imageCredentials`。
> 请参考 [先决条件](../../getting-started/Quickstarts/prerequisites) 部分获取更多信息。

## 使用 Dockerfile 构建和运行无服务器应用

如果你已经为你的应用创建了一个 `Dockerfile`，比如这个 [Go 应用](https://github.com/OpenFunction/samples/tree/main/apps/buildah/go)，你可以像 [这样](https://github.com/OpenFunction/samples/blob/main/apps/buildah/go/sample-go-app.yaml) 以无服务器的方式构建和运行这个应用：

1. 创建示例 Go 无服务器应用

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: sample-go-app
  namespace: default
spec:
  build:
    builder: openfunction/buildah:v1.23.1
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: buildah
    srcRepo:
      revision: main
      sourceSubPath: apps/buildah/go
      url: https://github.com/OpenFunction/samples.git
  image: openfunctiondev/sample-go-app:v1
  imageCredentials:
    name: push-secret
  serving:
    template:
      containers:
        - imagePullPolicy: IfNotPresent
          name: function
    triggers:
      http:
        port: 8080
  version: v1.0.0
  workloadRuntime: OCIContainer
EOF
```

2. 检查应用状态
你可以通过 kubectl get functions.core.openfunction.io -w 检查无服务器应用的状态：
```shell
kubectl get functions.core.openfunction.io -w
NAME                    BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                   AGE
sample-go-app           Succeeded    Running        builder-jgnzp   serving-q6wdp   http://sample-go-app.default.svc.cluster.local/           22m
```

3. 访问这个应用
一旦 `BUILDSTATE` 变成 `Succeeded`，`SERVINGSTATE` 变成 `Running`，你就可以通过 `ADDRESS` 字段中的地址访问这个 Go 无服务器应用：
```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://sample-go-app.default.svc.cluster.local
```

> [这里](https://github.com/OpenFunction/samples/tree/main/apps/buildah/java)你可以找到一个 Java 无服务器应用（带有 Dockerfile）的示例。

## 不使用 Dockerfile 构建和运行无服务器应用

如果你有一个没有 Dockerfile 的应用，比如这个 [Java 应用](https://github.com/buildpacks/samples/tree/main/apps/java-maven)，你也可以像这个 Java 应用 那样以无服务器的方式构建和运行你的应用:

1. 创建示例 Java 无服务器应用

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: sample-java-app-buildpacks
  namespace: default
spec:
  build:
    builder: cnbs/sample-builder:alpine
    srcRepo:
      revision: main
      sourceSubPath: apps/java-maven
      url: https://github.com/buildpacks/samples.git
  image: openfunction/sample-java-app-buildpacks:v1
  imageCredentials:
    name: push-secret
  serving:
    template:
      containers:
        - imagePullPolicy: IfNotPresent
          name: function
          resources: {}
    triggers:
      http:
        port: 8080
  version: v1.0.0
  workloadRuntime: OCIContainer

EOF
```

2. 检查应用状态
你可以通过 kubectl get functions.core.openfunction.io -w 检查无服务器应用的状态：
```shell
kubectl get functions.core.openfunction.io -w
NAME                                 BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                                AGE
sample-java-app-buildpacks           Succeeded    Running        builder-jgnzp   serving-q6wdp   http://sample-java-app-buildpacks.default.svc.cluster.local/           22m
```

3. 访问这个应用
一旦 `BUILDSTATE` 变成 `Succeeded`，`SERVINGSTATE` 变成 `Running`，你就可以通过 `ADDRESS` 字段中的地址访问这个 Java 无服务器应用：
```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://sample-java-app-buildpacks.default.svc.cluster.local
```