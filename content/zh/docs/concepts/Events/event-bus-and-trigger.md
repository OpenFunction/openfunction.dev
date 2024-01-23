---
title: "使用 EventBus 和 Trigger"
linkTitle: "使用 EventBus 和 Trigger"
weight: 4520
description:
---

本文档提供了一个示例，说明如何使用 EventBus 和 Trigger。

## 先决条件

- 您需要创建一个作为目标函数的函数以被触发。有关更多详细信息，请参见 [创建函数](../event-source#create-a-function)。
- 您需要创建一个 Kafka 集群。有关更多详细信息，请参见 [创建 Kafka 集群](../event-source#create-a-kafka-cluster)。

## 部署 NATS 流服务器

运行以下命令部署 NATS 流服务器。本文档使用 `nats://nats.default:4222` 作为 NATS 流服务器的访问地址，`stan` 作为集群 ID。有关更多信息，请参见 [NATS Streaming (STAN)](https://github.com/nats-io/k8s/tree/main/helm/charts/stan#tldr)。

```shell
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
helm install nats nats/nats
helm install stan nats/stan --set stan.nats.url=nats://nats:4222
```

## 创建 OpenFuncAsync 运行时函数

1. 使用以下内容创建目标函数的配置文件（例如，`openfuncasync-function.yaml`），该函数由 Trigger CRD 触发并打印接收到的消息。

   ```yaml
   apiVersion: core.openfunction.io/v1beta2
   kind: Function
   metadata:
     name: trigger-target
   spec:
     version: "v1.0.0"
     image: openfunctiondev/v1beta1-trigger-target:latest
     serving:
       scaleOptions:
         keda:
           scaledObject:
             pollingInterval: 15
             minReplicaCount: 0
             maxReplicaCount: 10
             cooldownPeriod: 30
           triggers:
             - type: stan
               metadata:
                 natsServerMonitoringEndpoint: "stan.default.svc.cluster.local:8222"
                 queueGroup: "grp1"
                 durableName: "ImDurable"
                 subject: "metrics"
                 lagThreshold: "10"
       triggers:
         dapr:
           - name: eventbus
             topic: metrics
       pubsub:
         eventbus:
           type: pubsub.natsstreaming
           version: v1
           metadata:
             - name: natsURL
               value: "nats://nats.default:4222"
             - name: natsStreamingClusterID
               value: "stan"
             - name: subscriptionType
               value: "queue"
             - name: durableSubscriptionName
               value: "ImDurable"
             - name: consumerID
               value: "grp1"
   ```

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f openfuncasync-function.yaml
   ```

## 创建 EventBus 和 EventSource

1. 使用以下内容创建 EventBus 的配置文件（例如，`eventbus.yaml`）。

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventBus
   metadata:
     name: default
   spec:
     natsStreaming:
       natsURL: "nats://nats.default:4222"
       natsStreamingClusterID: "stan"
       subscriptionType: "queue"
       durableSubscriptionName: "ImDurable"
   ```

2. 使用以下内容创建 EventSource 的配置文件（例如，`eventsource.yaml`）。

   {{% alert title="注意" color="success" %}}

   通过 `spec.eventBus` 设置事件总线的名称。

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: my-eventsource
   spec:
     logLevel: "2"
     eventBus: "default"
     kafka:
       sample-two:
         brokers: "kafka-server-kafka-brokers.default.svc.cluster.local:9092"
         topic: "events-sample"
         authRequired: false
   ```

3. 运行以下命令应用这些配置文件。

   ```shell
   kubectl apply -f eventbus.yaml
   kubectl apply -f eventsource.yaml
   ```

4. 运行以下命令检查结果。

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME             EVENTBUS   SINK   STATUS
   my-eventsource   default           Ready

   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   62m

   $ kubectl get components
   NAME                                                 AGE
   serving-9689d-component-ebfes-my-eventsource-cmcbw   46m
   serving-9689d-component-esc-kafka-sample-two-l99cg   46m
   serving-dxrhd-component-eventbus-t65q7               13m
   serving-zwlj4-component-ebft-my-trigger-4925n        100s
   ```

   {{% alert title="注意" color="success" %}}

   在使用事件总线的情况下，EventSource 控制器的工作流程描述如下：

   1. 创建一个名为 `my-eventsource` 的 EventSource 自定义资源。
   2. 检索并重新组织 EventBus 的配置，包括 EventBus 名称（在此示例中为 `default`）和与 EventBus 关联的 Dapr 组件的名称。
   3. 创建一个名为 `serving-xxxxx-component-ebfes-my-eventsource-xxxxx` 的 Dapr 组件，使 EventSource 能够与事件总线关联。
   4. 创建一个名为 `serving-xxxxx-component-esc-kafka-sample-two-xxxxx` 的 Dapr 组件，使 EventSource 能够与事件源关联。
   5. 创建一个名为 `serving-xxxxx-deployment-v100-xxxxx` 的 Deployment，用于处理事件。

   {{% /alert %}}

## 创建 Trigger

1. 使用以下内容创建 Trigger 的配置文件（例如，`trigger.yaml`）。

   {{% alert title="注意" color="success" %}}

   - 通过 `spec.eventBus` 设置与 Trigger 关联的事件总线。
   - 通过 `spec.inputs` 设置事件输入源。
   - 这是一个简单的触发器，它从名为 `default` 的 EventBus 中收集事件。当它从 EventSource `my-eventsource` 中检索到一个 `sample-two` 事件时，它触发一个名为 `function-sample-serving-qrdx8-ksvc-fwml8` 的 Knative 服务，并同时将事件发送到事件总线的 `metrics` 主题。

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: Trigger
   metadata:
     name: my-trigger
   spec:
     logLevel: "2"
     eventBus: "default"
     inputs:
       inputDemo:
         eventSource: "my-eventsource"
         event: "sample-two"
     subscribers:
       - condition: inputDemo
         topic: "metrics"
   ```

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f trigger.yaml
   ```

3. 运行以下命令检查结果。

   ```shell
   $ kubectl get triggers.events.openfunction.io
   NAME         EVENTBUS   STATUS
   my-trigger   default    Ready

   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   62m

   $ kubectl get components
   NAME                                                 AGE
   serving-9689d-component-ebfes-my-eventsource-cmcbw   46m
   serving-9689d-component-esc-kafka-sample-two-l99cg   46m
   serving-dxrhd-component-eventbus-t65q7               13m
   serving-zwlj4-component-ebft-my-trigger-4925n        100s
   ```

   {{% alert title="注意" color="success" %}}

   在使用事件总线的情况下，Trigger 控制器的工作流程如下：

   1. 创建一个名为 `my-trigger` 的 Trigger 自定义资源。
   2. 检索并重新组织 EventBus 的配置，包括 EventBus 名称（在此示例中为 `default`）和与 EventBus 关联的 Dapr 组件的名称。
   3. 创建一个名为 `serving-xxxxx-component-ebft-my-trigger-xxxxx` 的 Dapr 组件，使 Trigger 能够与事件总线关联。
   4. 创建一个名为 `serving-xxxxx-deployment-v100-xxxxx` 的 Deployment，用于处理触发任务。

   {{% /alert %}}

## 创建事件生产者

1. 使用以下内容创建事件生产者配置文件（例如，`events-producer.yaml`）。

   ```yaml
   apiVersion: core.openfunction.io/v1beta2
   kind: Function
   metadata:
     name: events-producer
   spec:
     version: "v1.0.0"
     image: openfunctiondev/v1beta1-bindings:latest
     serving:
       template:
         containers:
           - name: function
             imagePullPolicy: Always
       triggers:
         dapr:
           - name: cron
             type: bindings.cron
       outputs:
         - dapr:
             name: kafka-server
             operation: "create"
       bindings:
         cron:
           type: bindings.cron
           version: v1
           metadata:
             - name: schedule
               value: "@every 2s"
         kafka-server:
           type: bindings.kafka
           version: v1
           metadata:
             - name: brokers
               value: "kafka-server-kafka-brokers:9092"
             - name: topics
               value: "events-sample"
             - name: consumerGroup
               value: "bindings-with-output"
             - name: publishTopic
               value: "events-sample"
             - name: authRequired
               value: "false"
   ```

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f events-producer.yaml
   ```

3. 运行以下命令观察目标异步函数的变化。

   ```shell
   $ kubectl get functions.core.openfunction.io
   NAME                                  BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
   trigger-target                        Skipped      Running                  serving-dxrhd                                         20m
   
   $ kubectl get po --watch
   NAME                                                     READY   STATUS              RESTARTS   AGE
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      0/2     Pending             0          0s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      0/2     Pending             0          0s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      0/2     ContainerCreating   0          0s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      0/2     ContainerCreating   0          2s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      1/2     Running             0          4s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      1/2     Running             0          4s
   serving-dxrhd-deployment-v100-xmrkq-785cb5f99-6hclm      2/2     Running             0          4s
   ```