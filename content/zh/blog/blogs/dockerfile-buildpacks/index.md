---
title: '再见 Dockerfile，拥抱新型镜像构建技术 Buildpacks'
linkTitle: "再见 Dockerfile，拥抱新型镜像构建技术 Buildpacks" 
date: 2022-01-04
weight: 100
---

> 作者：米开朗基杨，方阗

云原生正在吞并软件世界，容器改变了传统的应用开发模式，如今研发人员不仅要构建应用，还要使用 Dockerfile 来完成应用的容器化，将应用及其依赖关系打包，从而获得更可靠的产品，提高研发效率。

随着项目的迭代，达到一定的规模后，就需要运维团队和研发团队之间相互协作。运维团队的视角与研发团队不同，他们对镜像的需求是**安全**和**标准化**。比如：

+ 不同的应用应该选择哪种基础镜像？
+ 应用的依赖有哪些版本？
+ 应用需要暴露的端口有哪些？

为了优化运维效率，提高应用安全性，研发人员需要不断更新 Dockerfile 来实现上述目标。同时运维团队也会干预镜像的构建，如果基础镜像中有 CVE 被修复了，运维团队就需要更新 Dockerfile，使用较新版本的基础镜像。总之，运维与研发都需要干预 Dockerfile，无法实现解耦。

为了解决这一系列的问题，涌现出了更加优秀的产品来构建镜像，其中就包括 [Cloud Native Buildpacks](https://buildpacks.io/) (СNB)。`CNB` 基于模块化提供了一种更加快速、安全、可靠的方式来构建符合 `OCI` 规范的镜像，实现了研发与运维团队之间的解耦。

在介绍 `CNB`  之前，我们先来阐述几个基本概念。

## 符合 OCI 规范的镜像

如今，容器运行时早就不是 `Docker` 一家独大了。为了确保所有的容器运行时都能运行任何构建工具生成的镜像，Linux 基金会与 Google，华为，惠普，IBM，Docker，Red Hat，VMware 等公司共同宣布成立开放容器项目（OCP），后更名为[开放容器倡议（OCI）](https://opencontainers.org/)。OCI 定义了围绕容器镜像格式和运行时的行业标准，给定一个 OCI 镜像，任何实现 [OCI 运行时标准](https://github.com/opencontainers/runtime-spec)的容器运行时都可以使用该镜像运行容器。

如果你要问 Docker 镜像与 OCI 镜像之间有什么区别，如今的答案是：**几乎没有区别**。有一部分旧的 Docker 镜像在 OCI 规范之前就已经存在了，它们被成为 Docker v1 规范，与 Docker v2 规范是不兼容的。而 Docker v2 规范捐给了 OCI，构成了 OCI 规范的基础。如今所有的容器镜像仓库、Kubernetes 平台和容器运行时都是围绕 OCI 规范建立的。

## 什么是 Buildpacks

Buildpacks 项目最早由 `Heroku` 在 2011 年发起, 被以 `Cloud Foundry` 为代表的 PaaS 平台广泛采用。

一个 **buildpack** 指的就是一个将源代码变成 PaaS 平台可运行的压缩包的程序，通常情况下，每个 buildpack 封装了单一的语言生态系统的工具链，例如 Ruby、Go、NodeJs、Java、Python 等都有专门的 buildpack。

你可以将 buildpack 理解成一坨脚本，这坨脚本的作用是将应用的可执行文件及其依赖的环境、配置、启动脚本等打包，然后上传到 Git 等仓库中，打好的压缩包被称为 **droplet**。

然后 Cloud Foundry 会通过调度器选择一个可以运行这个应用的虚拟机，然后通知这个机器上的 Agent 下载应用压缩包，按照 buildpack 指定的启动命令，启动应用。

到了 2018 年 1 月，`Pivotal` 和 `Heroku` 联合发起了 [Cloud Native Buildpacks(CNB)](https://buildpacks.io/) 项目，并在同年 10 月让这个项目进入了 `CNCF`。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311515749.png)

2020 年 11 月，CNCF 技术监督委员会（TOC）[投票决定将 CNB 从沙箱项目晋升为孵化项目](https://www.cncf.io/blog/2020/11/18/toc-approves-cloud-native-buildpacks-from-sandbox-to-incubation/)。是时候好好研究一下 CNB 了。

## 为什么需要 Cloud Native Buildpacks

Cloud Native Buildpacks(CNB) 可以看成是**基于云原生的 Buildpacks 技术**，它支持现代语言生态系统，对开发者屏蔽了应用构建、部署的细节，如选用哪种操作系统、编写适应镜像操作系统的处理脚本、优化镜像大小等等，并且会产出 OCI 容器镜像，可以运行在任何兼容 OCI 镜像标准的集群中。CNB 还拥抱了很多更加云原生的特性，例如[跨镜像仓库的 blob 挂载和镜像层级 rebasing](https://docs.docker.com/registry/spec/api/)。

由此可见 CNB 的镜像构建方式更加标准化、自动化，与 Dockerfile 相比，Buildpacks 为构建应用提供了更高层次的抽象，**Buildpacks 对 OCI 镜像构建的抽象，就类似于 Helm 对 Deployment 编排的抽象**。

2020 年 10 月，Google Cloud 开始宣布全面支持 Buildpacks，包含 Cloud Run、Anthos 和 Google Kubernetes Engine (GKE)。目前 IBM Cloud、Heroku 和 Pivital 等公司皆已采用 Buildpacks，如果不出意外，其他云供应商很快就会效仿。

Buildpacks 的优点：

+ 针对同一构建目的的应用，不用重复编写构建文件（只需要使用一个 Builder）。
+ 不依赖 Dockerfile。
+ 可以根据丰富的元数据信息（buildpack.toml）轻松地检查到每一层（buildpacks）的工作内容。
+ 在更换了底层操作系统之后，不需要重新改写镜像构建过程。
+ 保证应用构建的安全性和合规性，而无需开发者干预。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311522597.png)

Buildpacks 社区还给出了一个表格来对比同类应用打包工具：

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311519659.png)

可以看到 Buildpacks 与其他打包工具相比，支持的功能更多，包括：缓存、源代码检测、插件化、支持 rebase、重用、CI/CD 多种生态。

## Cloud Native Buildpacks 工作原理

Cloud Native Buildpacks 主要由 3 个组件组成： `Builder`、`Buildpack` 和 `Stack`。

### Buildpack

Buildpack 本质是一个可执行单元的集合，一般包括检查程序源代码、构建代码、生成镜像等。一个典型的 Buildpack 需要包含以下三个文件：

+ **buildpack.toml** – 提供 buildpack 的元数据信息。
+ **bin/detect** – 检测是否应该执行这个 buildpack。
+ **bin/build** – 执行 buildpack 的构建逻辑，最终生成镜像。

### Builder

Buildpacks 会通过“检测”、“构建”、“输出”三个动作完成一个构建逻辑。通常为了完成一个应用的构建，我们会使用到多个 Buildpacks，那么 `Builder` 就是一个构建逻辑的集合，包含了构建所需要的所有组件和运行环境的镜像。

我们通过一个假设的流水线来尝试理解 Builder 的工作原理：

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311536638.png)

+ 最初，我们作为应用的开发者，准备了一份应用源代码，这里我们将其标识为 “0”。
+ 然后应用 “0” 来到了第一道工序，我们使用 **Buildpacks1** 对其进行加工。在这个工序中，**Buildpacks1** 会检查应用是否具有 “0” 标识，如果有，则进入构建过程，即为应用标识添加 “1”，使应用标识变更为 “01”。
+ 同理，第二道、第三道工序也会根据自身的准入条件判断是否需要执行各自的构建逻辑。

在这个例子中，应用满足了三道工序的准入条件，所以最终输出的 OCI 镜像的内容为 “01234” 的标识。

对应到 Buildpacks 的概念中，`Builders` 就是 Buildpacks 的有序组合，包含一个基础镜像叫 `build image`、一个 `lifecycle` 和对另一个基础镜像 `run image` 的应用。Builders 负责将应用源代码构建成应用镜像（app image）。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311556268.png)

`build image` 为 **Builders** 提供基础环境（例如 带有构建工具的 Ubuntu Bionic OS 镜像），而 `run image` 在运行时为**应用镜像**（app image）提供基础环境。`build image` 和 `run image` 的组合被称为 [Stack](https://buildpacks.io/docs/concepts/components/stack)。

### Stack

上面提到，`build image` 和 `run image` 的组合被称为 Stack，也就是说，它定义了 Buildpacks 的执行环境和最终应用的基础镜像。

**你可以将 `build image` 理解为 Dockerfile 多阶段构建中第一阶段的 base 镜像，将 `run image` 理解为第二阶段的 base 镜像。**

----

上述 3 个组件都是以 Docker 镜像的形式存在，并且提供了非常灵活的配置选项，还拥有控制所生成镜像的每一个 layer 的能力。结合其强大的 [caching](https://buildpacks.io/docs/app-developer-guide/using-cache-image/) 和 [rebasing](https://buildpacks.io/docs/concepts/operations/rebase/) 能力，定制的组件镜像可以被多个应用重复利用，并且每一个 layer 都可以根据需要单独更新。

----

`Lifecycle` 是 **Builder** 中最重要的概念，它将由应用源代码到镜像的构建步骤抽象出来，完成了对整个过程的编排，并最终产出应用镜像。下面我们单独用一个章节来介绍 Lifecycle。

### 构建生命周期（Lifecyle）

Lifecycle 将所有 Buildpacks 的探测、构建过程抽离出来，分成两个大的步骤聚合执行：Detect 和 Build。这样一来就降低了 Lifecycle 的架构复杂度，便于实现自定义的 Builder。

除了 Detect 和 Build 这两个主要步骤，Lifecycle 还包含了一些额外的步骤，我们一起来解读。

#### Detect

我们之前提到，在 Buildpack 中包含了一个用于探测的 `/bin/detect` 文件，那么在 `Detect` 过程中，Lifecycle 会指导所有 Buildpacks 中的 `/bin/detect` 按顺序执行，并从中获取执行结果。

那么 Lifecycle 把 `Detect` 和 `Build` 分开后，又是怎么维系这两个过程中的关联关系呢？

Buildpacks 在 Detect 和 Build 阶段，通常都会告知在自己这个过程中会需要哪些前提，以及自己会提供哪些结果。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311624331.png)

在 Lifecycle 中，提供了一个叫做 `Build Plan` 的结构体用于存放每个 Buildpack 的所需物和产出物。

```go
type BuildPlanEntry struct { 
    Providers `toml:“providers”`     
    Requires  `toml:"requires"` 
```

同时，Lifecycle 也规定，只有当所有产出物都匹配有一个对应的所需物时，这些 Buildpacks 才能组合成一个 Builder。

#### Analysis

Buildpacks 在运行中会创建一些目录，在 Lifecycle 中这些目录被称为 `layer`。那么为了这些 `layer` 中，有一些是可以作为缓存提供给下一个 Buildpacks 使用的，有一些则是需要在应用运行时起作用的，还有的则是需要被清理掉。怎么才能更灵活地控制这些 `layer` ？

Lifecycle 提供了三个开关参数，用于表示每一个 `layer` 期望的处理方式：

+ **launch** 表示这个 layer 是否将在应用运行时起作用。
+ **build** 表示这个 layer 是否将在后续的构建过程中被访问。
+ **cache** 则表示这个 layer 是否将作为缓存。

之后，Lifecycle 再根据一个关系矩阵来判断 layer 的最终归宿。我们也可以简单的理解为，**Analysis 阶段为构建、应用运行提供了缓存**。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311631009.png)

#### Build

`Build` 阶段会利用 `Detect` 阶段产出的 `build plan`，以及环境中的元数据信息，配合保留至本阶段的 layers，对应用源码执行 Buildpacks 中的构建逻辑。最终生成可运行的应用工件。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311635266.png)

#### Export

Export 阶段比较好理解，在完成了上述构建之后，我们需要将最后的构建结果产出为一个 OCI 标准镜像，这样一来，这个 App 工件就可以运行在任何兼容 OCI 标准的集群中。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311638802.png)

#### Rebase

在 CNB 的设计中，最后 app 工件实际是运行在 stack 的 run image 之上的。可以理解为 run image 以上的工件是一个整体，它与 run image 以 ABI(application binary interface) 的形式对接，这就使得这个工件可以灵活切换到另一个 run image 上。

这个动作其实也是 Lifecycle 的一部分，叫做 **rebase**。在构建镜像的过程中也有一次 **rebase**，发生在 app 工件由 build image 切换到 run image 上。

![](https://pek3b.qingstor.com/kubesphere-community/images/202201041504352.png)

这种机制也是 CNB 对比 Dockerfile 最具优势的地方。比如在一个大型的生产环境中，如果容器镜像的 OS 层出现问题，需要更换镜像的 OS 层，那么针对不同类型的应用镜像就需要重写他们的 dockerfile 并验证新的 dockerfile 是否可行，以及新增加的层与已存在的层之间是否有冲突，等等。而使用 CNB 只需要做一次 rebase 即可，简化了大规模生产中镜像的升级工作。

----

以上就是关于 CNB 构建镜像的流程分析，总结来说：

+ Buildpacks 是最小构建单元，执行具体的构建操作；
+ Lifecycle 是 CNB 提供的镜像构建生命周期接口；
+ Builder 是若干 Buildpacks 加上 Lifecycle 以及 stack 形成的具备特定构建目的的构建器。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311642856.png)

再精减一下：

+ build image + run image = stack
+ stack(build image) + buildpacks + lifecycle = builder
+ stack(run image) + app artifacts = app

那么现在问题来了，这个工具怎么使用呢？

### Platform

这时候就需要一个 Platform，Platform 其实是 Lifecycle 的执行者。它的作用是将 Builder 作用于给定的源代码上，完成 Lifecycle 的指令。

![](https://pek3b.qingstor.com/kubesphere-community/images/202112311649597.png)

在这个过程中，Builder 会将源代码构建为 app，这个时候 app 是在 `build image` 中的。这个时候根据 Lifecycle 中的 rebase 接口，底层逻辑是是用 [ABI(application binary interface)](https://en.wikipedia.org/wiki/Application_binary_interface) 将 app 工件从 `build image` 转换到 `run image` 上。这就是最后的 OCI 镜像。

常用的 Platform 有 Tekton 和 CNB 的 [Pack](https://buildpacks.io/docs/tools/pack/)。接下来我们将使用 Pack 来体验如何使用 Buildpacks 构建镜像。

## 安装 Pack CLI 工具

目前 Pack CLI 支持 Linux、MacOS 和 Windows 平台，以 Ubuntu 为例，安装命令如下：

```bash
$ sudo add-apt-repository ppa:cncf-buildpacks/pack-cli
$ sudo apt-get update
$ sudo apt-get install pack-cli
```

查看版本：

```bash
$ pack version
0.22.0+git-26d8c5c.build-2970
```

> 注意：在使用 Pack 之前，需要先安装并运行 Docker。

目前 Pack CLI 只支持 Docker，不支持其他容器运行时（比如 Containerd 等）。但 Podman 可以通过一些 hack 来变相支持，以 Ubuntu 为例，大概步骤如下：

先安装 podman。

```bash
$ . /etc/os-release
$ echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
$ curl -L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/Release.key" | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get -y upgrade
$ sudo apt-get -y install podman
```

然后启用 Podman Socket。

```bash
$ systemctl enable --user podman.socket
$ systemctl start --user podman.socket
```

指定 `DOCKER_HOST` 环境变量。

```bash
$ export DOCKER_HOST="unix://$(podman info -f "{{.Host.RemoteSocket.Path}}")"
```

最终就可以实现在 Podman 容器运行时中使用 Pack 来构建镜像。详细配置步骤可参考 [Buildpacks 官方文档](https://buildpacks.io/docs/app-developer-guide/building-on-podman/)。

## 使用 Pack 构建 OCI 镜像

安装完 Pack 之后，我们可以通过 CNB 官方提供的 [samples](https://github.com/buildpacks/samples) 加深对 Buildpacks 原理的理解。这是一个 Java 示例，构建过程中无需安装 JDK、运行 Maven 或其他构建环境，Buildpacks 会为我们处理好这些。

首先克隆示例仓库：

```bash
$ git clone https://github.com/buildpacks/samples.git
```

后面我们将使用 bionic 这个 Builder 来构建镜像，先来看下该 Builder 的配置：

```toml
$ cat samples/builders/bionic/builder.toml
# Buildpacks to include in builder
[[buildpacks]]
id = "samples/java-maven"
version = "0.0.1"
uri = "../../buildpacks/java-maven"

[[buildpacks]]
id = "samples/kotlin-gradle"
version = "0.0.1"
uri = "../../buildpacks/kotlin-gradle"

[[buildpacks]]
id = "samples/ruby-bundler"
version = "0.0.1"
uri = "../../buildpacks/ruby-bundler"

[[buildpacks]]
uri = "docker://cnbs/sample-package:hello-universe"

# Order used for detection
[[order]]
[[order.group]]
id = "samples/java-maven"
version = "0.0.1"

[[order]]
[[order.group]]
id = "samples/kotlin-gradle"
version = "0.0.1"

[[order]]
[[order.group]]
id = "samples/ruby-bundler"
version = "0.0.1"

[[order]]
[[order.group]]
id = "samples/hello-universe"
version = "0.0.1"

# Stack that will be used by the builder
[stack]
id = "io.buildpacks.samples.stacks.bionic"
run-image = "cnbs/sample-stack-run:bionic"
build-image = "cnbs/sample-stack-build:bionic"
```

builder.toml 文件中完成了对 Builder 的定义，配置结构可以划分为 3 个部分：

+ **[[buildpacks]]** 语法标识用于定义 Builder 所包含的 Buildpacks。
+ **[[order]]** 用于定义 Builder 所包含的 Buildpacks 的执行顺序。
+ **[[stack]]** 用于定义 Builder 将运行在哪个基础环境之上。

我们可以使用这个 builder.toml 来构建自己的 builder 镜像：

```bash
$ cd samples/builders/bionic

$ pack builder create cnbs/sample-builder:bionic --config builder.toml
284055322776: Already exists
5b7c18d5e17c: Already exists
8a0af02bbad1: Already exists
0aa0fb9222a5: Download complete
3d56f4bc2c9a: Already exists
5b7c18d5e17c: Already exists
284055322776: Already exists
8a0af02bbad1: Already exists
a967314b5694: Already exists
a00d148009e5: Already exists
dbb2c49b44e3: Download complete
53a52c7f9926: Download complete
0cceee8a8cb0: Download complete
c238db6a02a5: Download complete
e925caa83f18: Download complete
Successfully created builder image cnbs/sample-builder:bionic
Tip: Run pack build <image-name> --builder cnbs/sample-builder:bionic to use this builder
```

接着，进入 samples/apps 目录，使用 pack 工具和 builder 镜像，完成应用的构建。当构建成功后，会产出一个名为 `sample-app` 的 OCI 镜像。

```bash
$ cd ../..
$ pack build --path apps/java-maven --builder cnbs/sample-builder:bionic sample-app
```

最后使用 Docker 运行这个 `sample-app` 镜像：

```bash
$ docker run -it -p 8080:8080 sample-app
```

访问 **http://localhost:8080**，如果一切正常，你可以在浏览器中看见如下的界面：

![](https://pek3b.qingstor.com/kubesphere-community/images/202201032322151.png)

现在我们再来观察一下之前构建的镜像：

```bash
$ docker images
REPOSITORY                               TAG              IMAGE ID       CREATED        SIZE
cnbs/sample-package                      hello-universe   e925caa83f18   42 years ago   4.65kB
sample-app                               latest           7867e21a60cd   42 years ago   300MB
cnbs/sample-builder                      bionic           83509780fa67   42 years ago   181MB
buildpacksio/lifecycle                   0.13.1           76412e6be4e1   42 years ago   16.4MB
```

镜像的创建时间竟然都是固定的时间戳：**42 years ago**。这是为什么呢？**如果时间戳不固定，每次构建镜像的 hash 值都是不同的，一旦 hash 值不一样，就不太容易判断镜像的内容是否相同了。使用固定的时间戳，就可以重复利用之前的构建过程中创建的 layers。**

## 总结

Cloud Native Buildpacks 代表了现代软件开发的一个重大进步，在大部份场景下相对于 Dockerfile 的好处是立杆见影的。虽然大型企业需要投入精力重新调整 CI/CD 流程或编写自定义 Builder，但从长远来看可以节省大量的时间和维护成本。

本文介绍了 Cloud Native Buildpacks(CNB) 的起源以及相对于其他工具的优势，并详细阐述了 CNB 的工作原理，最后通过一个简单的示例来体验如何使用 CNB 构建镜像。后续的文章将会介绍如何创建自定义的 Builder、Buildpack、Stack，以及函数计算平台（例如，[OpenFunction](https://github.com/OpenFunction/OpenFunction/)、Google Cloud Functions）如何利用 CNB  提供的 S2I 能力，实现从用户的函数代码到最终应用的转换过程。