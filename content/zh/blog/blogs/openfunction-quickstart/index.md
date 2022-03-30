---
title: '云原生 FaaS 平台 OpenFunction 入门教程'
linkTitle: "云原生 FaaS 平台 OpenFunction 入门教程" 
date: 2022-03-30
weight: 100
---

![](https://pek3b.qingstor.com/kubesphere-community/images/202203291741315.png)

> OpenFunction 0.6.0 上周已经正式发布了，带来了许多值得注意的功能，包括函数插件、函数的分布式跟踪、控制自动缩放、HTTP 函数触发异步函数等。同时，异步运行时定义也被重构了。核心 API 也已经从 v1alpha1 升级到 v1beta1。<br>
官宣链接🔗：https://openfunction.dev/blog/2022/03/25/announcing-openfunction-0.6.0-faas-observability-http-trigger-and-more/

近年来，随着**无服务器计算**的兴起，出现了很多非常优秀的 **Serverless** 开源项目，其中比较杰出的有 Knative 和 OpenFaaS。但 **Knative Serving** 仅仅能运行应用，还不能运行函数，而 Serverless 的核心是函数计算，也就是 FaaS，因此比较遗憾；**OpenFaaS** 虽然很早就出圈了，但技术栈过于老旧，不能满足现代化函数计算平台的需求。

[OpenFunction](https://github.com/OpenFunction/OpenFunction) 便是这样一个现代化的云原生 FaaS（函数即服务）框架，它引入了很多非常优秀的开源技术栈，包括 Knative、Tekton、Shipwright、Dapr、KEDA 等，这些技术栈为打造新一代开源函数计算平台提供了无限可能：

+ Shipwright 可以在函数构建的过程中让用户自由选择和切换镜像构建的工具，并对其进行抽象，提供了统一的 API；
+ Knative 提供了优秀的同步函数运行时，具有强大的自动伸缩能力；
+ KEDA 可以基于更多类型的指标来自动伸缩，更加灵活；
+ Dapr 可以将不同应用的通用能力进行抽象，减轻开发分布式应用的工作量。

![](https://pek3b.qingstor.com/kubesphere-community/images/202203291802577.png)

本文不打算讲一些非常高深的理论，作为刚跨进 Serverless 门槛的用户，更需要的是如何快速上手，以便对函数计算有一个感性的认知，在后续使用的过程中，咱们再慢慢理解其中的架构和设计。

本文将会带领大家快速部署和上手 OpenFunction，并通过一个 demo 来体验同步函数是如何运作的。

## OpenFunction CLI 介绍

OpenFunction 从 0.5 版本开始使用全新的命令行工具 [ofn](https://github.com/OpenFunction/cli) 来安装各个依赖组件，它的功能更加全面，支持一键部署、一键卸载以及 Demo 演示的功能。用户可以通过设置相应的参数自定义地选择安装各个组件，同时可以选择特定的版本，使安装更为灵活，安装进程也提供了实时展示，使得界面更为美观。它支持的组件和其依赖的 Kubernetes 版本如下：

| Components             | Kubernetes 1.17 | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20+ |
| :--------------------- | :-------------- | :-------------- | :-------------- | :--------------- |
| Knative Serving        | 0.21.1          | 0.23.3          | 0.25.2          | 1.0.1            |
| Kourier                | 0.21.0          | 0.23.0          | 0.25.0          | 1.0.1            |
| Serving Default Domain | 0.21.0          | 0.23.0          | 0.25.0          | 1.0.1            |
| Dapr                   | 1.5.1           | 1.5.1           | 1.5.1           | 1.5.1            |
| Keda                   | 2.4.0           | 2.4.0           | 2.4.0           | 2.4.0            |
| Shipwright             | 0.6.1           | 0.6.1           | 0.6.1           | 0.6.1            |
| Tekton Pipelines       | 0.23.0          | 0.26.0          | 0.29.0          | 0.30.0           |
| Cert Manager           | 1.5.4           | 1.5.4           | 1.5.4           | 1.5.4            |
| Ingress Nginx          | na              | na              | 1.1.0           | 1.1.0            |

<center>表一 OpenFunction 使用的第三方组件依赖的 Kubernetes 版本</center>
<br>
ofn 的安装参数 `ofn install` 解决了 OpenFunction 和 Kubernetes 的兼容问题，会自动根据 Kubernetes 版本选择兼容组件进行安装，同时提供多种参数以供用户选择。

| 参数             | 功能                                                      |
| :--------------- | :-------------------------------------------------------- |
| --all            | 用于安装 OpenFunction 及其所有依赖。                      |
| --async          | 用于安装 OpenFunction 的异步运行时（Dapr & Keda）。       |
| --cert-manager * | 用于安装 Cert Manager。                                   |
| --dapr *         | 用于安装 Dapr。                                           |
| --dry-run        | 用于提示当前命令所要安装的组件及其版本。                  |
| --ingress  *     | 用于安装 Ingress Nginx。                                  |
| --keda  *        | 用于安装 Keda。                                           |
| --knative        | 用于安装 Knative Serving（以Kourier为默认网关）           |
| --region-cn      | 针对访问 gcr.io 或 github.com 受限的用户。                |
| --shipwright *   | 用于安装 ShipWright。                                     |
| --sync           | 用于安装 OpenFunction Sync Runtime（待支持）。            |
| --upgrade        | 在安装时将组件升级到目标版本。                            |
| --verbose        | 显示粗略信息。                                            |
| --version        | 用于指定要安装的 OpenFunction 的版本。（默认为 "v0.6.0"） |
| --timeout        | 设置超时时间。默认为5分钟。                               |

<center>表二 install 命令参数列表</center>

## 使用 OpenFunction CLI 部署 OpenFunction

有了命令行工具 ofn 之后，OpenFunction 部署起来非常简单。首先需要安装 ofn，以 amd64 版本的 Linux 为例，仅需两步即可：

1、下载 ofn

```bash
$ wget -c  https://github.com/OpenFunction/cli/releases/download/v0.5.1/ofn_linux_amd64.tar.gz -O - | tar -xz
```

2、为 ofn 赋予权限并移动到 `/usr/local/bin/` 文件夹下。

```bash
$ chmod +x ofn && mv ofn /usr/local/bin/
```

安装好 ofn 之后，仅需一步即可完成 OpenFunction 的安装。虽然使用 `--all` 选项可以安装所有组件，但我知道大部分小伙伴的真实需求是不想再额外装一下 Ingress Controller 的，这个也好办，我们可以直接指定需要安装的组件，排除 ingress，命令如下：

```bash
$ ofn install --knative --async --shipwright --cert-manager --region-cn
Start installing OpenFunction and its dependencies.
The following components will be installed:
+------------------+---------+
| COMPONENT        | VERSION |
+------------------+---------+
| OpenFunction     | 0.6.0   |
| Keda             | 2.4.0   |
| Dapr             | 1.5.1   |
| Shipwright       | 0.6.1   |
| CertManager      | 1.5.4   |
| Kourier          | 1.0.1   |
| DefaultDomain    | 1.0.1   |
| Knative Serving  | 1.0.1   |
| Tekton Pipelines | 0.30.0  |
+------------------+---------+
 ✓ Dapr - Completed!
 ✓ Keda - Completed!
 ✓ Knative Serving - Completed!
 ✓ Shipwright - Completed!
 ✓ Cert Manager - Completed!
 ✓ OpenFunction - Completed!
🚀 Completed in 2m47.901328069s.

 ██████╗ ██████╗ ███████╗███╗   ██╗
██╔═══██╗██╔══██╗██╔════╝████╗  ██║
██║   ██║██████╔╝█████╗  ██╔██╗ ██║
██║   ██║██╔═══╝ ██╔══╝  ██║╚██╗██║
╚██████╔╝██║     ███████╗██║ ╚████║
 ╚═════╝ ╚═╝     ╚══════╝╚═╝  ╚═══╝

███████╗██╗   ██╗███╗   ██╗ ██████╗████████╗██╗ ██████╗ ███╗   ██╗
██╔════╝██║   ██║████╗  ██║██╔════╝╚══██╔══╝██║██╔═══██╗████╗  ██║
█████╗  ██║   ██║██╔██╗ ██║██║        ██║   ██║██║   ██║██╔██╗ ██║
██╔══╝  ██║   ██║██║╚██╗██║██║        ██║   ██║██║   ██║██║╚██╗██║
██║     ╚██████╔╝██║ ╚████║╚██████╗   ██║   ██║╚██████╔╝██║ ╚████║
╚═╝      ╚═════╝ ╚═╝  ╚═══╝ ╚═════╝   ╚═╝   ╚═╝ ╚═════╝ ╚═╝  ╚═══╝
```

虽然本文演示的是同步函数，但这里把异步运行时也装上了，如果你不需要，可以把 `--async` 这个参数去掉，不影响本文的实验。

安装完成后，会创建这几个 namespace：

```bash
$ kubectl get ns
NAME                              STATUS   AGE
cert-manager                      Active   17m
dapr-system                       Active   4m34s
io                                Active   3m31s
keda                              Active   4m49s
knative-serving                   Active   4m41s
kourier-system                    Active   3m57s
openfunction                      Active   3m37s
shipwright-build                  Active   4m26s
tekton-pipelines                  Active   4m50s
```

每个 namespace 对应上面安装的各个组件。目前 OpenFunction 的 Webhook 需要使用 **CertManager** 来验证 API 访问，后续我们会去掉这个依赖，不再需要安装 **CertManager**。

## 自定义域名后缀

Knative Serving 目前使用 `Kourier` 作为入口网关，由于我们没有部署 Ingress Controller，所以我们访问函数只有 `Kourier` 这一个入口。

[Kourier](https://github.com/knative-sandbox/net-kourier) 是一个基于 **Envoy Proxy** 的轻量级网关，是专门对于 Knative Serving 服务访问提供的一个网关实现。关于 Envoy 控制平面的细节本文不作赘述，感兴趣的可以去阅读 Kourier 官方文档和源码。这里我们只需要知道 Kourier 会为函数访问提供一个入口，这个访问入口是通过域名来提供的，我们要做的工作就是将相关域名解析到 Kourier 的 ClusterIP。

Kourier 默认创建了两个 Service：

```bash
$ kubectl -n kourier-system get svc
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kourier            LoadBalancer   10.233.7.202   <pending>     80:31655/TCP,443:30980/TCP   36m
kourier-internal   ClusterIP      10.233.47.71   <none>        80/TCP                       36m
```

只需要将与函数访问相关域名解析到 `10.233.47.71` 即可。

虽然每个函数的域名都是不同的，但域名后缀是一样的，可以通过泛域名解析来实现解析与函数相关的所有域名。Kourier 默认的域名后缀是 `example.com`，通过 Knative 的 ConfigMap `config-domain` 来配置：

```yaml
$ kubectl -n knative-serving get cm config-domain -o yaml
apiVersion: v1
data:
  _example: |
    ################################
    #                              #
    #    EXAMPLE CONFIGURATION     #
    #                              #
    ################################

    # This block is not actually functional configuration,
    # but serves to illustrate the available configuration
    # options and document them in a way that is accessible
    # to users that `kubectl edit` this config map.
    #
    # These sample configuration options may be copied out of
    # this example block and unindented to be in the data block
    # to actually change the configuration.

    # Default value for domain.
    # Although it will match all routes, it is the least-specific rule so it
    # will only be used if no other domain matches.
    example.com: |

    # These are example settings of domain.
    # example.org will be used for routes having app=nonprofit.
    example.org: |
      selector:
        app: nonprofit

    # Routes having the cluster domain suffix (by default 'svc.cluster.local')
    # will not be exposed through Ingress. You can define your own label
    # selector to assign that domain suffix to your Route here, or you can set
    # the label
    #    "networking.knative.dev/visibility=cluster-local"
    # to achieve the same effect.  This shows how to make routes having
    # the label app=secret only exposed to the local cluster.
    svc.cluster.local: |
      selector:
        app: secret
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

将其中的 `_example` 对象删除，添加一个默认域名（例如 `openfunction.dev`），最终修改结果如下：

```yaml
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

## 配置集群域名解析

为了便于在 Kubernetes 的 Pod 中访问函数，可以对 Kubernetes 集群的 CoreDNS 进行改造，使其能够对域名后缀 `openfunction.dev` 进行泛解析，需要在 CoreDNS 的配置中添加一段内容：

```caddy
        template IN A openfunction.dev {
          match .*\.openfunction\.dev
          answer "{{ .Name }} 60 IN A 10.233.47.71"
          fallthrough
        }
```

修改完成后的 CoreDNS 配置如下：

```bash
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

## 同步函数 demo 示例

配置完域名解析后，接下来可以运行一个同步函数的示例来验证一下。[OpenFunction 官方仓库提供了多种语言的同步函数示例](https://github.com/OpenFunction/samples/tree/main/functions/knative)：

![](https://pek3b.qingstor.com/kubesphere-community/images/202203291626247.png)

这里我们选择 Go 语言的函数示例，先来看一下最核心的部署清单：

```yaml
# function-sample.yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v2.0.0"
  image: "openfunctiondev/sample-go-func:latest"
  imageCredentials:
    name: push-secret
  port: 8080 # default to 8080
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "HelloWorld"
      FUNC_CLEAR_SOURCE: "true"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/knative/hello-world-go"
      revision: "main"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    runtime: "knative"
```

`Function` 是由 CRD 定义的一个 CR，用来将函数转换为最终运行的应用。这个例子里面包含了两个组件：

+ **build** : 通过 Shipwright 选择不同的镜像构建工具，最终将应用构建为容器镜像；
+ **Serving** : 通过 Serving CRD 将应用部署到不同的运行时中，可以选择同步运行时或异步运行时。这里选择的是同步运行时 knative。

国内环境由于不可抗因素，可以通过 `GOPROXY` 从公共代理镜像中快速拉取所需的依赖代码，只需在部署清单中的 `build` 阶段添加一个环境变量 `FUNC_GOPROXY` 即可：

```yaml
# function-sample.yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v2.0.0"
  image: "openfunctiondev/sample-go-func:latest"
  imageCredentials:
    name: push-secret
  port: 8080 # default to 8080
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "HelloWorld"
      FUNC_CLEAR_SOURCE: "true"
      FUNC_GOPROXY: "https://proxy.golang.com.cn,direct"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/knative/hello-world-go"
      revision: "main"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    runtime: "knative"
```

在创建函数之前，需要先创建一个 secret 来存储 Docker Hub 的用户名和密码：

```bash
$ REGISTRY_SERVER=https://index.docker.io/v1/ REGISTRY_USER=<your_registry_user> REGISTRY_PASSWORD=<your_registry_password>
$ kubectl create secret docker-registry push-secret \
    --docker-server=$REGISTRY_SERVER \
    --docker-username=$REGISTRY_USER \
    --docker-password=$REGISTRY_PASSWORD
```

下面通过 kubectl 创建这个 Function：

```bash
$ kubectl apply -f function-sample.yaml
```

查看 Function 运行状况：

```bash
$ kubectl get function
NAME              BUILDSTATE   SERVINGSTATE   BUILDER         SERVING   URL   AGE
function-sample   Building                    builder-6ht76                   5s
```

目前正处于 Build 阶段，builder 的名称是 `builder-6ht76`。查看 builder 的运行状态：

```bash
$ kubectl get builder
NAME            PHASE   STATE      REASON   AGE
builder-6ht76   Build   Building            50s
```

这个 builder 会启动一个 Pod 来构建镜像：

```bash
$ kubectl get pod
NAME                                     READY   STATUS     RESTARTS   AGE
builder-6ht76-buildrun-jvtwk-vjlgt-pod   2/4     NotReady   0          2m11s
```

这个 Pod 中包含了 4 个容器：

+ **step-source-default** : 拉取源代码；

  ![](https://pek3b.qingstor.com/kubesphere-community/images/202203291651337.png)

+ **step-prepare** : 设置环境变量；

  ![](https://pek3b.qingstor.com/kubesphere-community/images/202203291652921.png)

+ **step-create** : 构建镜像；

  ![](https://pek3b.qingstor.com/kubesphere-community/images/202203291654385.png)

+ **step-results** : 输出镜像的 digest。

再次查看函数状态：

```bash
$ kubectl get function
NAME              BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         URL                                              AGE
function-sample   Succeeded    Running        builder-6ht76   serving-6w4rn   http://openfunction.io/default/function-sample   6m
```

已经由之前的 **Building** 状态变成了 **Runing** 状态。

这里的 URL 我们无法直接访问，因为没有部署 Ingress Controller。不过我们可以通过其他方式来访问，Kourier 把每个访问入口抽象为一个 CR 叫 **ksvc**，每一个 **ksvc** 对应一个函数的访问入口，可以看下目前有没有创建 ksvc：

```bash
$ kubectl get ksvc
NAME                       URL                                                        LATESTCREATED                   LATESTREADY                     READY   REASON
serving-6w4rn-ksvc-k4x29   http://serving-6w4rn-ksvc-k4x29.default.openfunction.dev   serving-6w4rn-ksvc-k4x29-v200   serving-6w4rn-ksvc-k4x29-v200   True
```

函数的访问入口就是 **http://serving-6w4rn-ksvc-k4x29.default.openfunction.dev**。由于在前面的章节中已经配置好了域名解析，这里可以启动一个 Pod 来直接访问该域名：

```bash
$ kubectl run curl --image=radial/busyboxplus:curl -i --tty
If you don't see a command prompt, try pressing enter.
[ root@curl:/ ]$
[ root@curl:/ ]$ curl http://serving-6w4rn-ksvc-k4x29.default.openfunction.dev/default/function-sample/World
Hello, default/function-sample/World!
[ root@curl:/ ]$ curl http://serving-6w4rn-ksvc-k4x29.default.openfunction.dev/default/function-sample/OpenFunction
Hello, default/function-sample/OpenFunction!
```

访问这个函数时会自动触发运行一个 Pod：

```bash
$ kubectl get pod
NAME                                                       READY   STATUS    RESTARTS   AGE
serving-6w4rn-ksvc-k4x29-v200-deployment-688d58bfb-6fvcg   2/2     Running   0          7s
```

这个 Pod 使用的镜像就是之前 build 阶段构建的镜像。事实上这个 Pod 是由 Deployment 控制的，在没有流量时，这个 Deployment 的副本数是 0。当有新的流量进入时，会先进入 Knative 的 Activator，Activator 接收到流量后会通知 Autoscaler（自动伸缩控制器），然后 Autoscaler 将 Deployment 的副本数扩展到 1，最后 Activator 会将流量转发到实际的 Pod 中，从而实现服务调用。这个过程也叫**冷启动**。

如果你不再访问这个入口，过一段时间之后，Deployment 的副本数就会被收缩为 0：

```bash
$ kubectl get deploy
NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
serving-6w4rn-ksvc-k4x29-v200-deployment   0/0     0            0           22m
```

## 总结

通过本文的示例，相信大家应该能够体会到一些函数计算的优势，它为我们带来了我们所期望的对业务场景快速拆解重构的能力。作为用户，只需要专注于他们的开发意图，编写函数代码，并上传到代码仓库，其他的东西不需要关心，不需要了解基础设施，甚至不需要知道容器和 Kubernetes 的存在。函数计算平台会自动为您分配好计算资源，并弹性地运行任务，只有当您需要访问的时候，才会通过扩容来运行任务，其他时间并不会消耗计算资源。