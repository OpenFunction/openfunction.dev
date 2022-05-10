---
title: '快速上手 OpenFunction Node.js 异步函数服务开发'
linkTitle: '快速上手 OpenFunction Node.js 异步函数服务开发'
date: 2022-05-08
weight: 100
---

10 多天前，“OpenFunction 顺利通过了云原生计算基金会 CNCF 技术监督委员会（TOC）的投票，正式进入 CNCF 沙箱（Sandbox）托管”。作为 OpenFunction 社区的一份子，非常期待能有更多开发者和合作伙伴参与到项目中来，共同建设和发展社区，“使 Serverless 函数与应用运行更简单”！同时，作为 Node.js 函数框架（Function Framework）目前的 Maintainer 之一，也想借此机会和大家分享一下 Node.js 函数框架最近的研发进展，特别是在 [0.4.1](https://www.npmjs.com/package/@openfunction/functions-framework/v/0.4.1) 版本中已经实现的对于异步函数的支持。

本文将从以下几方面来介绍 Node.js 函数框架目前的研发进展和之后的工作展望。
![](outline.jpg)

## 同步函数当前状态简述

> 一句话简述：支持 Express 形态的 “请求-响应” 函数调研，同时也支持接收 CloudEvents 标准定义的事件数据。

在 0.4.1 版本中，我们基于 GCP（Google Cloud Platform）Function Framework 对 Node.js 函数框架了进行了重建，在同步函数方面基本完整的保留了 GCP Node.js 函数框架的现有能力。

首先，最经典的 Express 形态的函数签名是必须支持的，这也是我们日常进行同步函数开发的主要形态。

```javascript
/**
 * Send "Hello, World!"
 * @param req https://expressjs.com/en/api.html#req
 * @param res https://expressjs.com/en/api.html#res
 */
export const helloWorld = (req, res) => {
  res.send('Hello, World!');
};
```

> - 您可以参考 [Quickstart: "Hello, World" on your local machine](https://www.npmjs.com/package/@openfunction/functions-framework/v/0.4.1#quickstart-hello-world-on-your-local-machine) 在本地编写并运行同步函数，并参考 [此样例](https://github.com/OpenFunction/samples/tree/main/functions/knative/hello-world-node/commonjs) 中的 Function 资源描述来在 Kubernetes 中进行部署和运行。
> - 关于 OpenFunction 框架的基础使用方式，您可以参考 [开源函数计算平台 OpenFunction 保姆级入门教程](https://mp.weixin.qq.com/s/zJSIDXbGuGnNcQE5pn_ERA) 文中的细节。

其次，[CloudEvents](https://cloudevents.io/) 作为云原生领域日益重要的事件数据（Event Data）描述标准，我们的同步函数也已支持接收 CloudEvents 标准定义的事件数据。您可以参考 [此文档](https://github.com/openfunction/functions-framework-nodejs/blob/HEAD/docs/cloudevents.md) 在本地构建函数并测试 CloudEvents 的接收处理。

```javascript
const functions = require('@openfunction/functions-framework');

functions.cloudEvent('helloCloudEvents', (cloudevent) => {
  console.log(cloudevent.specversion);
  console.log(cloudevent.type);
  console.log(cloudevent.source);
  console.log(cloudevent.subject);
  console.log(cloudevent.id);
  console.log(cloudevent.time);
  console.log(cloudevent.datacontenttype);
});
```

> **同步函数的版本迭代计划**
>
> [OpenFunction 0.6.0](https://mp.weixin.qq.com/s/UfSQ2TVoNUpa36DdZR3X2A) 为其同步函数增加了 Dapr 输出绑定（Output Binding），使异步函数通过 HTTP 同步函数进行触发成为了可能（例如由 Knative 运行时支持的同步函数现在可以与由 Dapr 输出绑定或 Dapr Pub/Sub 中间件进行交互，异步函数将被同步函数发送的事件所触发）。**我们将在下一个 Node.js 函数框架的迭代版本中提供此项同步函数增强能力的支持。**
> ![http-trigger-openfunction](/images/docs/en/blogs/http-trigger-openfunction.jpg)

## 异步函数快速上手指北

> 一句话简述：现已支持通过 Node.js 异步函数接收和调用 Dapr 输入/输出绑定（Input/Output Binding）和发布/订阅（Pub/Sub）构建块的能力。

### 示例环境准备

> 为了方便同时展示 “输入输出绑定” 和 “发布订阅” 这两个功能，我们在以下两个示例中采用了 MQTT 这个同时支持这两种异步消息使用模式的组件，所以需要先在 Kubernetes 环境中部署一个 MQTT 中间件服务。

我们在这里选用 EMQ 公司的开源 MQTT 中间件 EMQX 作为我们示例运行基础组件，它支持通过 Helm 方式进行部署（参见 [通过 Helm3 在 Kubernetes 上部署 EMQX 4.0 集群](https://www.emqx.com/zh/blog/rapidly-deploy-emqx-clusters-on-kubernetes-via-helm)）：

```bash
helm repo add emqx https://repos.emqx.io/charts
helm repo update
```

```bash
$ helm search repo emqx

NAME               CHART VERSION APP VERSION DESCRIPTION
emqx/emqx          4.4.3         4.4.3       A Helm chart for EMQX
emqx/emqx-ee       4.4.3         4.4.3       A Helm chart for EMQ X
emqx/emqx-operator 1.0.4         1.1.6       A Helm chart for EMQX Operator Controller
```

```bash
helm install emqx emqx/emqx --set replicaCount=1 --set service.type=NodePort 
```

部署完成后，您可用检查 EMQX StatefulSet 和 Service 的状态，请确保它们都进入了运行状态：

```bash
$ kubectl get sts -o wide

NAME   READY   AGE   CONTAINERS   IMAGES
emqx   1/1     11d   emqx         emqx/emqx:4.4.3
```

```bash
$ kubectl get svc

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                      AGE
emqx            NodePort    10.233.52.80    <none>        1883:32296/TCP,8883:32089/TCP,8081:32225/TCP,8083:32740/TCP,8084:31394/TCP,18083:30460/TCP   11d
emqx-headless   ClusterIP   None            <none>        1883/TCP,8883/TCP,8081/TCP,8083/TCP,8084/TCP,18083/TCP,4370/TCP                              11d                                                                                  31d
```

记住这里 `1883`端口对应的 NodePort 端口 `32296`，我们会在后面的示例实验过程中用到（NodePort 在每次部署的时候都会变化，以实际部署时为准）。

同时您可用通过访问 EMQX 自带的 Dashboard 来确认部署成功（端口：`18083`对应的 NodePort 端口；用户名：`admin`；默认密码：`public`）。
![EMQX Dashboard 运行界面示例](eqmx-dashboard.png)

> 关于 MQTT 协议以及 EMQ X Broker 中间件的行业应用，可以参见我们在 KubeSphere 社区直播中的分享《 [MQTT 及车联网场景应用](https://kubesphere.com.cn/live/mqtt1230-live/) 》。
>
> 为了方便后续执行 MQTT 消息的发布和监测，我们可以下载同样来自于 EMQ 公司的 [MQTT X](https://mqttx.app/) 桌面客户端工具备用。您也可以使用任意 MQTT 客户端来完成相关操作。

### 示例函数编写

下面我们编写一个非常简单的异步函数作为示例，整个项目只需要两个文件：`index.mjs`，`package.json`。

```javascript
// 同步函数入口
export const tryKnative = (req, res) => {
  res.send(`Hello, ${req.query.u || 'World'}!`);
};

// 异步函数入口
export const tryAsync = (ctx, data) => {
  console.log('Data received: %o', data);
  ctx.send(data);
};
```

我们先来看 `index.mjs` 这个主文件，其中有两个关键点需要留意：

- 异步函数签名：异步函数使用 `function (ctx, data)` 作为函数签名，其中：
  - `ctx` 包含了执行的 [上下文数据](https://github.com/OpenFunction/functions-framework-nodejs/blob/master/src/openfunction/function_context.ts)，同时具有一个 `send(data, ouput?)` 方法用于向 Dapr 全部（或特定）的输出绑定或发布通道发送数据
  - `data` 是从 Dapr 的输入绑定或订阅通道接收的数据
- 主文件可以同时包含同步和异步函数入口：这一点是 Go 函数框架做不到的，后文会介绍如何使用这个特性（如果大家熟悉 Node.js 动态脚本语言的性质就不难理解）

```json
{
  "main": "index.mjs",
  "scripts": {
    "start": "functions-framework --target=tryKnative"
  },
  "dependencies": {
    "@openfunction/functions-framework": "^0.4.1"
  }
}
```

元数据文件 `package.json` 可以定义的非常简单，核心就是两部分：

- `main`：指定主函数文件（注意从 Node.js 15.3.0 开始，ES Modules 就已可以稳定使用，因此推荐直接使用 `.mjs` 后缀来标记 ESM 格式文件）
- `scripts``dependencies`：这些主要是为了方便本地测试而准备的，非必需，但推荐也安排上
  - 关于 `functions-framework` 命令行的可惜参数，可以参考 [Configure the Functions Framework](https://www.npmjs.com/package/@openfunction/functions-framework/v/0.4.1#configure-the-functions-framework)

### 示例镜像准备

> OpenFunction 自带 Shipwirght / Tekton 可以实施 Cloud Native Buildpacks 的 OCI 镜像打包，本地打包更适合网络环境比较糟糕（或 GitHub 访问困难）的集群。

这里我们推荐使用跨平台但 [Pack](https://buildpacks.io/docs/tools/pack/) 工具来实施本地打包，安装之后的使用命令也非常简单（如下所示），镜像生成后推送至您指定的仓库备用。

```bash
pack build -B openfunction/builder-node:v2-16.13 -e FUNC_NAME=tryKnative -p src <image-repo>/<image-name>:<tag>
```
>
> - `-B openfunction/builder-node:v2-16.13`：必填，16.13 是目前 OpenFunction 可以使用的最新的 Node.js 环境 Builder，未来会迭代更新的版本（撰文是最新版本为 18）
> - `-e FUNC_NAME=tryKnative`：必填，设置默认的入口函数，建议选择一个最基础的同步函数作为入口，不妨碍后续异步函数使用（详见后文）
> - `-p src`：默认是使用当前目录，但建议把源文件和 OpenFunction CR 文件分层或分目录存在，用通过设置文件路径来使用

### 示例：MQTT 输入输出绑定

> 任务目标：我们的服务需要从 `in` 主题接收一条输入消息，并将其作为输出数据发送给 `out` 主题消息通道中。

万事俱备，让我们先来看如何使用异步函数联通 MQTT 的输入输出，下面是一个 Function CR 的示例（参见 [Function CRD](https://openfunction.dev/docs/reference/component-reference/function-spec/)定义）。

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: sample-node-async-bindings
spec:
  version: v2.0.0
  image: '<image-repo>/<image-name>:<tag>'
  serving:
    # default to knative
    runtime: async
    annotations:
      # default to "grpc"
      dapr.io/app-protocol: http
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    params:
      # default to FUNC_NAME value
      FUNCTION_TARGET: tryAsync
    inputs:
      - name: mqtt-input
        component: mqtt-in
    outputs:
      - name: mqtt-output
        component: mqtt-out
        operation: create
    bindings:
      mqtt-in:
        type: bindings.mqtt
        version: v1
        metadata:
          - name: consumerID
            value: '{uuid}'
          - name: url
            value: tcp://admin:public@emqx:1883
          - name: topic
            value: in
      mqtt-out:
        type: bindings.mqtt
        version: v1
        metadata:
          - name: consumerID
            value: '{uuid}'
          - name: url
            value: tcp://admin:public@emqx:1883
          - name: topic
            value: out
```

让我们逐一解读一下 OpenFunction Serving 阶段的几个重要配置项及其内容：

- `serving.runtime`：异步函数使用 `async`，默认值是 `knative` 指代同步函数
- `serving.annoations`：必须设置注释 `dapr.io/app-protocol: http`，原因是目前 Node.js 函数框架是通过 HTTP 与 Dapr Sidecar 进行双向连接的，而 OpenFunction 默认使用 gRPC 协议和函数框架通信（虽然 Dapr 默认是 HTTP，有点绕 🤦‍♂️）
- `serving.params`：通过这个入口可以设置运行时的环境变量，于是我们便可以通过 `FUNCTION_TARGET: tryAsync` 在此动态指定函数入口，可以是任一已被模块导出的函数
- `serving.inputs/outputs`：输入/输出绑定及其操作的设置，参见 [Function CRD - DaprIO](https://openfunction.dev/docs/reference/component-reference/function-spec/#daprio) 的各个可用字段描述
- `serving.bindings`：这部分定义 Dapr 的绑定组件，每个组件对象的键（如 `mqtt-in``mqtt-out`）需要被 `serving.inputs/outputs` 准确引用，而值的部分则完全参考 Dapr 官方文档的 [Component specs - Bindings](https://docs.dapr.io/reference/components-reference/supported-bindings/) 来填写即可（注意当前 OpenFunction 0.6.0 对应使用的 Dapr 版本为 1.5.1）

#### 部署并确认函数运行

应用该 YAML 后，可查看函数及其对应的 Pod 状态：

```bash
$ kubectl apply -f async-bindings.yaml 
function.core.openfunction.io/sample-node-async-bindings created

$ kubectl get fn
NAME                         BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL      AGE
sample-node-async-bindings   Skipped      Running                  serving-8f7xc            140m

$ kubectl get po
NAME                                                   READY   STATUS    RESTARTS   AGE
...                                                 
serving-8f7xc-deployment-v200-l78xc-564c6b5bf7-vksg7   2/2     Running   0          141m

```

进而可以进一步查看 Pod 中 `function` 函数容器的日志，可以得到如下启动信息：

```bash
$ kubectl logs -c function serving-8f7xc-deployment-v200-l78xc-564c6b5bf7-vksg7
2022-05-04T13:06:18.505Z common:options ℹ️ Context loaded: <...>
[Dapr-JS] Listening on 8080
[Dapr API][PubSub] Registering 0 PubSub Subscriptions
[Dapr-JS] Letting Dapr pick-up the server (Maximum 60s wait time)
[Dapr-JS] - Waiting till Dapr Started (#0)
[Dapr API][PubSub] Registered 0 PubSub Subscriptions
[Dapr-JS] Server Started
```

#### 使用 MQTT X 测试输入输出

打开 MQTT X 并使用之前记录的 EMQX 服务 NodePort 端口 `32296` 创建连接，并添加一个 `out/#`的订阅。之后向 `in`主题发送内容 `{"msg": "hello"}`即可得到如下界面效果：
![使用 MQTT X 测试异步函数 MQTT 输入输出绑定](mqttx-in-out.png)
 如上图所示，可以看到消息向 `in` 主题发送，通过异步函数的转发，在 `out` 主题中被接收。也可以进一步从函数的容器日志中查看到输出 `Data received: { msg: 'hello' }`。

```bash
$ kubectl logs -c function serving-8f7xc-deployment-v200-l78xc-564c6b5bf7-vksg7
2022-05-04T13:06:18.505Z common:options ℹ️ Context loaded: <...>
[Dapr-JS] Listening on 8080
[Dapr API][PubSub] Registering 0 PubSub Subscriptions
[Dapr-JS] Letting Dapr pick-up the server (Maximum 60s wait time)
[Dapr-JS] - Waiting till Dapr Started (#0)
[Dapr API][PubSub] Registered 0 PubSub Subscriptions
[Dapr-JS] Server Started
Data received: { msg: 'hello' }
```

### 示例：MQTT 订阅及发布

> 任务目标：我们的服务需要订阅 `sub` 主题，并将接收到的消息发布到 `pub` 主题中。**（请特别留意本示例和前一示例的异同）**

发布订阅 MQTT 的功能在 OpenFunction 和 Dapr 框架的支持下也是非常简单，Function 的定义几乎和上一部分的绑定一样，下面我们挑选其中有变化的部分加以说明：

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: sample-node-async-pubsub
spec:
  version: v2.0.0
  image: '<image-repo>/<image-name>:<tag>'
  serving:
    # default to knative
    runtime: async
    annotations: ...
    template: ...
    params: ...
    inputs:
      - name: mqtt-sub
        component: mqtt-pubsub
        topic: sub
    outputs:
      - name: mqtt-pub
        component: mqtt-pubsub
        topic: pub
    pubsub:
      # Dapr MQTT PubSub: https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-mqtt/
      mqtt-pubsub:
        type: pubsub.mqtt
        version: v1
        metadata:
          - name: consumerID
            value: '{uuid}'
          - name: url
            value: tcp://admin:public@emqx:1883
          - name: qos
            value: 1
```

- `serving.inputs/outputs`：这部分基本和绑定示例中的一样，需要特别注意的是 `topic` 字段是发布订阅模式下专属的一个字段，用于定义发布订阅的主题
- `serving.pubsub`：这部分和 `serving.bindings` 也基本类似，这部分定义 Dapr 的发布订阅组件，每个组件对象的键（如 `mqtt-pubsub`）需要被 `serving.inputs/outputs` 准确引用，而值的部分则完全参考 Dapr 官方文档的 [Component specs - Pub/sub brokers](https://docs.dapr.io/reference/components-reference/supported-pubsub/) 来填写即可

#### 部署并确认函数运行

这部分和上一示例中的步骤基本一致，我们需要确仍 Pod 的运行状态及日志，特别是从函数日志上可以清新的看到如 `Registered 1 PubSub Subscriptions`的输出。

```bash
$ kubectl apply -f async-pubsub.yaml 
function.core.openfunction.io/sample-node-async-pubsub created

$ kubectl get fn
NAME                         BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL          AGE
sample-node-async-bindings   Skipped      Running                  serving-8f7xc                16h
sample-node-async-pubsub     Skipped      Running                  serving-2qfkl                16h

$ kubectl get po
NAME                                                   READY   STATUS    RESTARTS   AGE
...
serving-2qfkl-deployment-v200-6cshf-57c8b5b8dd-ztmbf   2/2     Running   0          16h
serving-8f7xc-deployment-v200-l78xc-564c6b5bf7-vksg7   2/2     Running   0          16h

$ kubectl logs -c function serving-2qfkl-deployment-v200-6cshf-57c8b5b8dd-ztmbf
2022-05-05T05:14:03.094Z common:options ℹ️ Context loaded: <...>
[Dapr-JS] Listening on 8080
[Dapr API][PubSub] Registering 1 PubSub Subscriptions
[Dapr-JS] Letting Dapr pick-up the server (Maximum 60s wait time)
[Dapr-JS] - Waiting till Dapr Started (#0)
[Dapr API][PubSub] Registered 1 PubSub Subscriptions
[Dapr-JS] Server Started

```

#### 使用 MQTT X 测试输入输出

在 MQTT X 中我们新建一个 `pub/#` 的订阅，之后向 `sub` 主题发布一段 JSON，可以得到如下的收发数据界面效果。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/99167/1651729251685-1b3205b1-b892-4c3f-8a1e-c3c7bcbf995d.png#clientId=u35635aed-3ef7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=867&id=u20e2ad01&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1734&originWidth=2260&originalType=binary&ratio=1&rotation=0&showTitle=false&size=463262&status=done&style=none&taskId=u02dd462f-4cf4-409a-979a-2b9aba3f12d&title=&width=1130)

大家可能已经注意到了，我们在发布订阅中发送的数据看似 “非常复杂”？没错！因为 Dapr 的发布和订阅功能默认都是使用 CloudEvents 数据格式来进行数据传输的（上图中的样例数据参见官方文档的 [Sending a custom CloudEvent](https://docs.dapr.io/developing-applications/building-blocks/pubsub/howto-publish-subscribe/#sending-a-custom-cloudevent) 部分的示例）。

但是，对于我们的函数框架，我们收到的数据是 CloudEvent 中被解析出来的 `data` 部分：如本例中的 `Data received: { orderId: '100' }` —— 也就是说 Dapr Sidecar 会处理数据负载的装箱和拆箱，对于函数开发者来说可以 “忽略” CloudEvent 数据包这部分的结构。

```bash
$ kubectl logs -c function serving-2qfkl-deployment-v200-6cshf-57c8b5b8dd-ztmbf
2022-05-05T05:14:03.094Z common:options ℹ️ Context loaded: <...>
[Dapr-JS] Listening on 8080
[Dapr API][PubSub] Registering 1 PubSub Subscriptions
[Dapr-JS] Letting Dapr pick-up the server (Maximum 60s wait time)
[Dapr-JS] - Waiting till Dapr Started (#0)
[Dapr API][PubSub] Registered 1 PubSub Subscriptions
[Dapr-JS] Server Started
Data received: { orderId: '100' }
```

> 以上两个示例的完整代码（含 Function Build 部分的参考 YAML）可以参见 OpenFunction 的在线样例库中的 [Node.js 样例](https://github.com/webup/openfunction-samples/tree/main/functions/node) 部分。

## 下一阶段的展望

> OpenFunction 0.6.0 的发布带来了许多值得关注的功能，包括函数插件、函数的分布式跟踪、控制自动缩放、HTTP 函数触发异步函数等。同时，异步运行时定义也被重构了。核心 API 也已经从 `v1alpha1` 升级到 `v1beta1`。

除了前文提到的 “HTTP 函数触发异步函数” 即将在下一个版本中进行实现。还有两个重要的功能也是下阶段的重点：

- **函数插件**：在 OpenFunction 的函数 CRD 中，允许用户定义在主体（Main）函数运行前/后执行的插件（Plugin）函数，并在函数运行时依靠函数框架保障插件的运行及其运行关系。您可以参见 [此案例](https://github.com/OpenFunction/samples/blob/cf5e42547ae67b45cf38dd125192a44c6e896131/functions/async/bindings/cron-input-kafka-output/cron-input-kafka-output.yaml#L6) 中的插件定义来初步了解。
- **函数可观测**：第二项重要的功能是 [使用 SkyWalking 为 OpenFunction 提供可观测能力](https://openfunction.dev/zh/docs/best-practices/skywalking-solution-for-openfunction/)。类似的，这些功能也需要函数框架的支持来使得 SkyWalking 可以正确的构建函数关系和追踪链路。

目前 OpenFunction Go 语言函数框架是完整支持上述两项功能的，我们期望在后续 Node.js 函数框架中也使能这两项能力。我们也已经在今年的 [开源之夏](https://mp.weixin.qq.com/s/spFdhhVaXN_ATMsRng-q3g) 活动中也开放了这些内容作为 [项目选题](https://github.com/kubesphere/community/blob/master/sig-advocacy-and-outreach/ospp-2022/openfunction-nodejs-function-framework-upgrade_zh-CN.md)，非常欢迎社区的同学们（字面意义的同学们）参与进来，一起共建我们的 OpenFunction 生态，让 Serverless 函数与应用运行更简单！
