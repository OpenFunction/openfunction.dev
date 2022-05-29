---
title: "快速上手"
linkTitle: "快速上手"
weight: 5
description: >
    本文将指导你如何在你的本地环境中开始构建 OpenFunction。
---

## 前提准备

---

如果你对开发控制器、管理器感兴趣，请参阅 [sample-controller](https://github.com/kubernetes/sample-controller) 和 [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)。

### Go

OpenFunction 基于 [Kubernetes](https://github.com/kubernetes/kubernetes)。它们都是用 [Go](https://go.dev/) 编写的。如果你没有 Go 开发环境，请先 [设置 Go 开发环境](https://go.dev/doc/code)。

| Kubernetes | 要求的 Go 版本 |
| ---------- | -------------- |
| 1.18+      | go>=1.12       |

> 提示：
>
> - 确保你的 GOPATH 和 PATH 已经按照 Go 环境说明进行了配置。
> - 在使用 MacOS 进行开发时，建议安装 [macOS GNU tools](https://www.topbug.net/blog/2013/04/14/install-and-use-gnu-command-line-tools-in-mac-os-x)。

### Docker

OpenFunction 组件通常在 Kubernetes 中以容器方式部署。如果你需要在 Kubernetes 集群中部署 OpenFunction 组件，你需要提前 [安装 Docker](https://docs.docker.com/install/)。

### 依赖管理

OpenFunction 使用 [Go Modules](https://github.com/golang/go/wiki/Modules) 管理依赖包。

## 制作镜像 & 运行

---

你可以通过修改 `cmd/Dockerfile` 来为你的本地环境制作 `openfunction` 镜像。

将镜像上传到你的个人镜像仓库后，在工作负载（Deployment）openfunction-controller-manager 中改变 `openfunction` 容器的镜像为你的个人镜像。

```shell
kubectl edit deployments.apps -n openfunction openfunction-controller-manager
```

保存后，Kubernetes 将自动应用新的镜像运行工作负载。