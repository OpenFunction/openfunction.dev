---
title: "安装"
linkTitle: "安装"
weight: 2100
description:
---

本文档将介绍如何安装 OpenFunction。

## 先决条件

- 您需要拥有一个 Kubernetes 集群。

- 您需要确保您的 Kubernetes 版本满足以下兼容性矩阵中描述的要求。

| OpenFunction Version | Kubernetes 1.21 | Kubernetes 1.22 | Kubernetes 1.23 | Kubernetes 1.24 | Kubernetes 1.25 | Kubernetes 1.26+ |
|----------------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| HEAD                 | N/A             | N/A             | √               |  √              | √               | √               |
| v1.2                 | N/A             | N/A             | √               |  √              | √               | √               |
| v1.1.x               | √               | √               | √               |  √              | √               | N/A             |
| v1.0.x               | √               | √               | √               |  √              | √               | N/A             |

## 安装 OpenFunction

现在您可以使用 helm charts 安装 OpenFunction 及其所有依赖项。
> `ofn` CLI 安装方法已经弃用。

如果您想在离线环境中安装 OpenFunction，请参考 [在离线环境中安装 OpenFunction](https://openfunction.dev/docs/reference/faq/#q-how-to-install-openfunction-in-an-offline-environment)

### 环境要求
- Kubernetes version: `>=v1.21.0-0`
- Helm version: `>=v3.6.3`

### 安装 OpenFunction helm charts 的步骤

1. 首先运行以下命令添加 OpenFunction chart 仓库:
   ```shell
   helm repo add openfunction https://openfunction.github.io/charts/
   helm repo update
   ```

2. 然后您有几个选项来设置 OpenFunction，您可以选择:

   - 安装所有组件:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction openfunction/openfunction -n openfunction
      ```
     
   - 安装所有组件和 [Revision Controller](https://openfunction.dev/docs/concepts/cicd/):
      ```shell
      kubectl create namespace openfunction
      helm install openfunction openfunction/openfunction -n openfunction --set revisionController.enable=true
      ```   

   - 只安装 Serving（不包括 build）:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.ShipwrightBuild.enabled=false --set global.TektonPipelines.enabled=false openfunction/openfunction -n openfunction
      ```
   
   - 只安装 Knative 同步运行时:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.Keda.enabled=false openfunction/openfunction -n openfunction
      ```
   
   - 只安装 OpenFunction 异步运行时:
      ```shell
      kubectl create namespace openfunction
      helm install openfunction --set global.Contour.enabled=false  --set global.KnativeServing.enabled=false openfunction/openfunction -n openfunction
      ```

   {{% alert title="Note" color="success" %}}

   关于如何使用 Helm 安装 OpenFunction 的更多信息，请参见 [使用 Helm 安装 OpenFunction](https://github.com/OpenFunction/charts#install-the-chart).

   {{% /alert %}}

3. Run the following command to verify OpenFunction is up and running:
   ```shell
   kubectl get po -n openfunction
   ```

## 卸载 OpenFunction

### Helm

如果您使用 Helm 安装了 OpenFunction，请运行以下命令来卸载 OpenFunction 及其依赖项。

```shell
helm uninstall openfunction -n openfunction
```
{{% alert title="Note" color="success" %}}

关于如何使用 Helm 卸载 OpenFunction 的更多信息，请参见 [使用 Helm 卸载 OpenFunction](https://github.com/OpenFunction/charts#uninstall-the-chart).

{{% /alert %}}

## 升级 OpenFunction

```shell
helm upgrade [RELEASE_NAME] openfunction/openfunction -n openfunction
```

使用 Helm v3，chart 创建的 CRD 默认不会更新，应手动更新。 另请参阅 [Helm 关于 CRD 的文档](https://helm.sh/docs/chart_best_practices/custom_resource_definitions)。

_参考 to [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) 以获取命令文档。_

### 将现有的发布升级到新版本

### 从 OpenFunction v0.6.0 升级到 OpenFunction v0.7.x

从 v0.6.0 升级到 0.7.x 有一个破坏性的变化，需要额外的手动操作。

#### 卸载 Chart

首先，您需要卸载旧的 `openfunction` release:
```shell
helm uninstall openfunction -n openfunction
```

确认组件命名空间已被删除，这需要一些时间：
```shell
kubectl get ns -o=jsonpath='{range .items[?(@.metadata.annotations.meta\.helm\.sh/release-name=="openfunction")]}{.metadata.name}: {.status.phase}{"\n"}{end}'
```

> 如果 knative-serving 命名空间长时间处于终止状态，尝试运行以下命令并删除 finalizers：
```shell
kubectl edit ingresses.networking.internal.knative.dev -n knative-serving
```

#### 升级 OpenFunction CRDs

```shell
kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/openfunction.yaml
```

#### 升级依赖组件的 CRDs

> 您只需要处理现有发布中包含的组件。
- knative-serving CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/knative-serving.yaml
    ```
- shipwright-build CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/shipwright-build.yaml
    ```
- tekton-pipelines CRDs
    ```shell
    kubectl apply -f https://openfunction.sh1a.qingstor.com/crds/v0.7.0/tekton-pipelines.yaml
    ```

#### 安装新 chart
```shell
helm repo update
helm install openfunction openfunction/openfunction -n openfunction
```

{{% alert title="Note" color="success" %}}

有关如何使用 Helm 升级 OpenFunction 的更多信息，请参见 [使用 Helm 升级 OpenFunction](https://github.com/OpenFunction/charts#upgrading-chart).

{{% /alert %}}