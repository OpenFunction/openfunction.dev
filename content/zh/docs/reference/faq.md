---
title: "常见问题解答"
linkTitle: "FAQ"
weight: 7100
description: 
---

本文档描述在使用OpenFunction时的常见问题解答。

## Q: 如何在OpenFunction中使用私有镜像仓库？

A: OpenFunction在构建阶段使用Shipwright（利用Tekton与Cloud Native Buildpacks集成）将用户函数打包到应用镜像中。

用户通常选择以不安全的方式访问私有镜像仓库，这在Cloud Native Buildpacks中尚不受支持。

我们提供以下解决方法，暂时绕过这个限制：

1. 使用IP地址而不是主机名作为私有镜像仓库的访问地址。
2. 运行Knative-runtime函数时，应该[跳过标签解析](https://knative.dev/docs/serving/configuration/deployment/#skipping-tag-resolution)。

参考链接：

[buildpacks/lifecycle#524](https://github.com/buildpacks/lifecycle/issues/524)

[buildpacks/tekton-integration#31](https://github.com/buildpacks/tekton-integration/issues/31)

## Q: 如何在不引入新的入口控制器的情况下访问Knative-runtime函数？

A: OpenFunction提供了函数可访问性的统一入口点，它基于Ingress Nginx实现。然而，对于一些用户来说，这是不必要的，引入新的入口控制器可能会影响当前集群。

一般来说，可访问的地址用于同步（Knative-runtime）函数。以下是解决此问题的两种方法：

- Magic DNS

  您可以按照[此指南](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#configure-dns)配置DNS。

- CoreDNS

  这类似于使用Magic DNS，不同之处在于DNS解析的配置放置在CoreDNS内部。假设用户已在`knative-serving`命名空间的`config-domain` ConfigMap下配置了一个名为"openfunction.dev"的域（如下所示）：

  ```shell
  $ kubectl -n knative-serving get cm config-domain -o yaml
  
  apiVersion: v1
  data:
    openfunction.dev: ""
  kind: ConfigMap
  metadata:
    annotations:
      knative.dev/example-checksum: 81552d0b
    labels:
      app.kubernetes.io/part-of: knative-serving
      app.kubernetes.io/version: 1.0.1
      serving.knative.dev/release: v1.0.1
    name: config-domain
    namespace: knative-serving
  ```

接下来，让我们为此域添加一个A记录。OpenFunction使用Kourier作为Knative Serving的默认网络层，该域名应该流向到Kourier。

  ```shell
  $ kubectl -n kourier-system get svc
  
  NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
  kourier            LoadBalancer   10.233.7.202   <pending>     80:31655/TCP,443:30980/TCP   36m
  kourier-internal   ClusterIP      10.233.47.71   <none>        80/TCP                       36m
  ```

然后，用户只需在CoreDNS中配置此通配符DNS解析，以解析集群中任何Knative服务的URL地址。

> 其中"10.233.47.71"是Service kourier-internal的地址。

  ```shell
  $ kubectl -n kube-system get cm coredns -o yaml
  
  apiVersion: v1
  data:
    Corefile: |
      .:53 {
          errors
          health
          ready
          template IN A openfunction.dev {
            match .*\.openfunction\.dev
            answer "{{ .Name }} 60 IN A 10.233.47.71"
            fallthrough
          }
          kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
          }
          hosts /etc/coredns/NodeHosts {
            ttl 60
            reload 15s
            fallthrough
          }
          prometheus :9153
          forward . /etc/resolv.conf
          cache 30
          loop
          reload
          loadbalance
      }
      ...
  ```

如果用户在集群外无法解析此函数的URL地址，请配置`hosts`文件如下：

> 其中"serving-sr5v2-ksvc-sbtgr.default.openfunction.dev"是从命令"kubectl get ksvc"中获取的URL地址。

  ```shell
  10.233.47.71 serving-sr5v2-ksvc-sbtgr.default.openfunction.dev
  ```

配置完成后，可以使用以下命令获取函数的URL地址。然后，您可以使用`curl`或浏览器触发该函数。

```shell
$ kubectl get ksvc

NAME                       URL
serving-sr5v

2-ksvc-sbtgr   http://serving-sr5v2-ksvc-sbtgr.default.openfunction.dev
```

## Q: 如何为函数启用和配置并发性？

A: OpenFunction将函数类型分为"[同步运行时](../../concepts/function/#the-sync-runtime)"和"[异步运行时](../../concepts/function/#the-async-runtime)"，基于正在处理的请求类型。这两种类型的函数由Knative Serving和Dapr + KEDA驱动。

因此，要启用和配置函数的并发性，您需要参考上述组件中的具体实现。

以下部分介绍了如何根据"[同步运行时](../../concepts/function/#the-sync-runtime)"和"[异步运行时](../../concepts/function/#the-async-runtime)"部分在OpenFunction中启用和配置函数的并发性。

### 同步运行时

您可以首先参考Knative Serving中的[此文档](https://knative.dev/docs/serving/autoscaling/concurrency/)，了解如何启用和配置并发性功能。

#### 软限制

您可以参考此[文档](https://knative.dev/docs/serving/autoscaling/concurrency/#soft-limit)中的`Global(ConfigMap)`和`Global(Operator)`部分，配置全局并发性功能。

而对于`Per Revision`，您可以像这样进行配置[此处](../../concepts/function_scaling_trigger/)。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
        autoscaling.knative.dev/target: "200"
```

#### 硬限制

OpenFunction目前不支持为`Per Revision`配置硬限制。您可以参考[此文档](https://knative.dev/docs/serving/autoscaling/concurrency/#hard-limit)中的`Global(ConfigMap)`和`Global(Operator)`部分，配置全局并发性功能。

#### 简而言之

简而言之，您可以为每个函数配置Knative Serving的与自动缩放相关的配置项，如下所示，只要它们可以作为***注释***传递，否则只能进行全局设置。

```yaml
# 在Knative Serving中的配置
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/<key>: "value"

# 在OpenFunction中的配置（推荐）
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
          <key>: "value"

# 替代方法
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    annotations:
      autoscaling.knative.dev/<key>: "value"
```

### 异步运行时

您可以首先参考Dapr中的[此文档](https://docs.dapr.io/operations/configuration/control-concurrency/)，了解如何启用和配置并发性功能。

与同步运行时的并发性配置相比，异步运行时的并发性配置更简单。

```yaml
# 在Dapr中的配置
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodesubscriber
  namespace: default
spec:
  template:
    metadata:
      annotations:
        dapr.io/app-max-concurrency: "value"

# 在OpenFunction中的配置（推荐）
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    annotations:
      dapr.io/app-max-concurrency: "value"
```

## Q: 如何为函数镜像构建过程创建源代码仓库凭据？

A: 在使用`spec.build.srcRepo.credentials`时，您可能会遇到类似`Unsupported type of credentials provided, either SSH private key or username/password is supported (exit code 110)`的错误，这意味着您正在使用不正确的Secret资源作为源代码仓库凭据。

OpenFunction目前基于[ShipWright](https://shipwright.io/)实现函数镜像构建框架，因此我们需要参考此[文档](https://shipwright.io/docs/build/authentication/#authentication-for-git)来设置正确的源代码仓库凭据。

## Q: 如何在离线环境中安装OpenFunction？

A: 您可以通过以下步骤在离线环境中安装和使用OpenFunction：

### 拉取Helm Chart

在可以访问GitHub的环境中拉取Helm Chart：

```shell
helm repo add openfunction https://openfunction.github.io/charts/
helm repo update
helm pull openfunction/openfunction
```

然后使用诸如scp之类的工具将Helm包复制到离线环境，例如：

```shell
scp openfunction-v1.0.0-v0.5.0.tgz <username>@<your-machine-ip>:/home/<username>/
```

### 同步镜像

您需要将这些镜像同步到您的私有镜像仓库：

```
# dapr
docker.io/daprio/dashboard:0.10.0
docker.io/daprio/dapr:1.8.3

# keda
openfunction/keda:2.8.1
openfunction/keda-metrics-apiserver:2.8.1

# contour
docker.io/bitnami/contour:1.21.1-debian-11-r5
docker.io/bitnami/envoy:1.22.2-debian-11-r6
docker.io/bitnami/nginx:1.21.6-debian-11-r10

# tekton-pipelines
openfunction/tektoncd-pipeline-cmd-controller:v0.37.2
openfunction/tektoncd-pipeline-cmd-kubeconfigwriter:v0.37.2
openfunction/tektoncd-pipeline-cmd-git-init:v0.37.2
openfunction/tektoncd-pipeline-cmd-entrypoint:v0.37.2
openfunction/tektoncd-pipeline-cmd-nop:v0.37.2
openfunction/tektoncd-pipeline-cmd-imagedigestexporter:v0.37.2
openfunction/tektoncd-pipeline-cmd-pullrequest-init:v0.37.2
openfunction/tektoncd-pipeline-cmd-pullrequest-init:v0.37.2
openfunction/tektoncd-pipeline-cmd-workingdirinit:v0.37.2
openfunction/cloudsdktool-cloud-sdk@sha256:27b2c22bf259d9bc1a291e99c63791ba0c27a04d2db0a43241ba0f1f20f4067f
openfunction/distroless-base@sha256:b16b57be9160a122ef048333c68ba205ae4fe1a7b7cc6a5b289956292ebf45cc
openfunction/tektoncd-pipeline-cmd-webhook:v0.37.2

# knative-serving
openfunction/knative.dev-serving-cmd-activator:v1.3.2
openfunction/knative.dev-serving-cmd-autoscaler:v1.3.2
openfunction/knative.dev-serving-cmd-queue:v1.3.2
openfunction/knative.dev-serving-cmd-controller:v1.3.2
openfunction/knative.dev-serving-cmd-domain-mapping:v1.3.2
openfunction/knative.dev-serving-cmd-domain-mapping-webhook:v1.3.2
openfunction/knative.dev-net-contour-cmd-controller:v1.3.0
openfunction/knative.dev-serving-cmd-default-domain:v1.3.2
openfunction/knative.dev-serving-cmd-webhook:v1.3.2

# shipwright-build
openfunction/shipwright-shipwright-build-controller:v0.10.0
openfunction/shipwright-io-build-git:v0.10.0
openfunction/shipwright-mutate-image:v0.10.0
openfunction/shipwright-bundle:v0.10.0
openfunction/shipwright-waiter:v0.10.0
openfunction/buildah:v1.23.3
openfunction/buildah:v1.28.0

# openfunction
openfunction/openfunction:v1.0.0
openfunction/kube-rbac-proxy:v0.8.0
openfunction/eventsource-handler:v4
openfunction/trigger-handler:v4
openfunction/dapr-proxy:v0.1.1
openfunction/revision-controller:v1.0.0
```

### 创建自定义值

在您的离线环境中创建 `custom-values.yaml` 文件：

```shell
touch custom-values.yaml
```

编辑 `custom-values.yaml`，添加以下内容：

```yaml
knative-serving:
  activator:
    activator:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-activator
  autoscaler:
    autoscaler:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-autoscaler
  configDeployment:
    queueSidecarImage:
      repository: <您的私有镜像仓库>/knative.dev-serving-cmd-queue
  controller:
    controller:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-controller
  domainMapping:
    domainMapping:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-domain-mapping
  domainmappingWebhook:
    domainmappingWebhook:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-domain-mapping-webhook
  netContourController:
    controller:
      image:
        repository: <您的私有镜像仓库>/knative.dev-net-contour-cmd-controller
  defaultDomain:
    job:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-default-domain
  webhook:
    webhook:
      image:
        repository: <您的私有镜像仓库>/knative.dev-serving-cmd-webhook
shipwright-build:
  shipwrightBuildController:
    shipwrightBuild:
      image:
        repository: <您的私有镜像仓库>/shipwright-shipwright-build-controller
      GIT_CONTAINER_IMAGE:
        repository: <您的私有镜像仓库>/shipwright-io-build-git
      MUTATE_IMAGE_CONTAINER_IMAGE:
        repository: <您的私有镜像仓库>/shipwright-mutate-image
      BUNDLE_CONTAINER_IMAGE:
        repository: <您的私有镜像仓库>/shipwright-bundle
      WAITER_CONTAINER_IMAGE:
        repository: <您的私有镜像仓库>/shipwright-waiter
tekton-pipelines:
  controller:
    tektonPipelinesController:
      image:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-controller
      kubeconfigWriterImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-kubeconfigwriter
      gitImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-git-init
      entrypointImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-entrypoint
      nopImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-nop
      imagedigestExporterImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-imagedigestexporter
      prImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-pullrequest-init
      workingdirinitImage:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-workingdirinit
      gsutilImage:
        repository: <您的私有镜像仓库>/cloudsdktool-cloud-sdk
        digest: sha256:27b2c22bf259d9bc1a291e99c63791ba0c27a04d2db0a43241ba0f1f20f4067f
      shellImage:
        repository: <您的私有镜像仓库>/distroless-base
        digest: sha256:b16b57be9160a122ef048333c68ba205ae4fe1a7b7cc6a5b289956292ebf45cc
  webhook:
    webhook:
      image:
        repository: <您的私有镜像仓库>/tektoncd-pipeline-cmd-webhook
keda:
  image:
    keda:
      repository: <您的私有镜像仓库>/keda
      tag: 2.8.1
    metricsApiServer:
      repository: <您的私有镜像仓库>/keda-metrics-apiserver
      tag: 2.8.1
dapr:
  global:
    registry: <您的私有镜像仓库>/daprio
    tag: '1.8.3'
contour:
  contour:
    image:
      registry: <您的私有镜像仓库>
      repository: <您的私有镜像仓库>/contour
      tag: 1.21.1-debian-11-r5
  envoy:
    image:
      registry: <您的私有镜像仓库>
      repository: <您的私有镜像仓库>/envoy
      tag: 1.22.2-debian-11-r6
  defaultBackend:
    image:
      registry: <您的私有镜像仓库>
      repository: <您的私有镜像仓库>/nginx
      tag: 1.21.6-debian-11-r10
```

请将 `<您的私有镜像仓库>` 替换为您的私有镜像仓库的地址。

### 安装 OpenFunction

在离线环境中运行以下命令尝试安装 OpenFunction：

```shell
kubectl create namespace openfunction
helm install openfunction openfunction-v1.0.0-v0.5.0.tgz -n openfunction -f custom-values.yaml
```

{{% alert title="注意" color="success" %}}

如果 `helm install` 命令卡住，可能是由于 job `contour-contour-cergen` 引起的。

运行以下命令确认 job 是否执行成功：

```shell
kubectl get job contour-contour-cergen -n projectcontour
```

如果 job 存在且 job 状态为完成，运行以下命令完成安装：

```shell
helm uninstall openfunction -n openfunction --no-hooks
helm install openfunction openfunction-v1.0.0-v0.5.0.tgz -n openfunction -f custom-values.yaml --no-hooks
```

{{% /alert %}}

### 补丁 ClusterBuildStrategy

如果您想在离线环境中构建 wasm 函数或使用 `buildah` 构建函数，运行以下命令补丁 `ClusterBuildStrategy`：

```shell
kubectl patch clusterbuildstrategy buildah --type='json' -p='[{"op": "replace", "path": "/spec/buildSteps/0/image", "value":"openfunction/buildah:v1.28.0"}]'
kubectl patch clusterbuildstrategy wasmedge --type='json' -p='[{"op": "replace", "path": "/spec/buildSteps/0/image", "value":"openfunction/buildah:v1.28.0"}]'
```

## Q: 如何在离线环境中构建和运行函数

A: 以 [Java 函数](https://github.com/OpenFunction/samples/tree/main/functions/knative/java) 为例，说明如何在离线环境中构建和运行函数：

- 将 `https://github.com/OpenFunction/samples.git` 同步到您的私有代码仓库

- 按照此 [prerequisites](https://openfunction.dev/docs/getting-started/quickstarts/prerequisites/) 文档创建 `push-secret` 和 `git-repo-secret`

- 将公共 maven 仓库更改为私有 maven 仓库：
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <groupId>dev.openfunction.samples</groupId>
      <artifactId>samples</artifactId>
      <version>1.0-SNAPSHOT</version>

      <properties>
          <maven.compiler.source>11</maven.compiler.source>
          <maven.compiler.target>11</maven.compiler.target>
      </properties>

      <repositories>
          <repository>
              <id>snapshots</id>
              <name>Maven snapshots</name>
                  <!--<url>https://s01.oss.sonatype.org/content/repositories/snapshots/</url>-->
                  <url>your private maven repository</url>
              <releases>
                  <enabled>false</enabled>
              </releases>
              <snapshots>
                  <enabled>true</enabled>
              </snapshots>
          </repository>
      </repositories>

      <dependencies>
          <dependency>
              <groupId>dev.openfunction.functions</groupId>
              <artifactId>functions-framework-api</artifactId>
              <version>1.0.0-SNAPSHOT</version>
          </dependency>
      </dependencies>

  </project>
  ```
  > 确保将更改提交到代码仓库。

- 将 `openfunction/buildpacks-java18-run:v1` 同步到您的私有镜像仓库

- 根据您的环境修改 `functions/knative/java/hello-world/function-sample.yaml`：
  ```yaml
  apiVersion: core.openfunction.io/v1beta2
  kind: Function
  metadata:
    name: function-http-java
  spec:
    version: "v2.0.0"
    image: "<your private image repository>/sample-java-func:v1"
    imageCredentials:
      name: push-secret
    build:
      builder: <your private image repository>/builder-java:v2-18
      params:
        RUN_IMAGE: "<your private image repository>/buildpacks-java18-run:v1"
      env:
        FUNC_NAME: "dev.openfunction.samples.HttpFunctionImpl"
        FUNC_CLEAR_SOURCE: "true"
      srcRepo:
        url: "https://<your private code repository>/OpenFunction/samples.git"
        sourceSubPath: "functions/knative/java"
        revision: "main"
        credentials:
         name: git-repo-secret
    serving:
      template:
        containers:
          - name: function # DO NOT change this
            imagePullPolicy: IfNotPresent
      triggers:
        http:
          port: 8080
  ```

  > 如果您的私有镜像仓库是不安全的，请参考 [以不安全的方式使用私有镜像仓库](#q-如何在openfunction中使用私有镜像仓库)

- 运行以下命令构建和运行函数：

  ```shell
  kubectl apply -f functions/knative/java/hello-world/function-sample.yaml
  ```