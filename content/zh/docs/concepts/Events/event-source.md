---
title: "使用 EventSource"
linkTitle: "使用 EventSource"
weight: 4510
description:
---

本文档提供了一个示例，说明如何使用事件源触发同步函数。

在此示例中，定义了一个 EventSource，用于同步调用，使用事件源（一个 Kafka 服务器）作为函数（一个 Knative 服务）的输入绑定。当事件源生成事件时，它将通过 `spec.sink` 配置调用函数并获取同步返回。

## 创建函数

使用以下内容创建一个函数作为 EventSource Sink。有关如何创建函数的更多信息，请参见 [创建同步函数](../../../getting-started/quickstarts/sync-functions)。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: sink
spec:
  version: "v1.0.0"
  image: "openfunction/sink-sample:latest"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    triggers:
      http:
        port: 8080
```

创建函数后，运行以下命令获取函数的 URL。

{{% alert title="注意" color="success" %}}

在函数的 URL 中，`openfunction` 是 Kubernetes 服务的名称，`io` 是运行 Kubernetes 服务的命名空间。有关更多信息，请参见 [服务的命名空间](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#namespaces-of-services)。

{{% /alert %}}

```shell
$ kubectl get functions.core.openfunction.io
NAME   BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
sink   Skipped      Running                  serving-4x5wh   https://openfunction.io/default/sink   13s
```

## 创建 Kafka 集群

1. 运行以下命令在默认命名空间中安装 [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator)。

   ```shell
   helm repo add strimzi https://strimzi.io/charts/
   helm install kafka-operator -n default strimzi/strimzi-kafka-operator
   ```

2. 使用以下内容创建一个文件 `kafka.yaml`。

   ```yaml
   apiVersion: kafka.strimzi.io/v1beta2
   kind: Kafka
   metadata:
     name: kafka-server
     namespace: default
   spec:
     kafka:
       version: 3.3.1
       replicas: 1
       listeners:
         - name: plain
           port: 9092
           type: internal
           tls: false
         - name: tls
           port: 9093
           type: internal
           tls: true
       config:
         offsets.topic.replication.factor: 1
         transaction.state.log.replication.factor: 1
         transaction.state.log.min.isr: 1
         default.replication.factor: 1
         min.insync.replicas: 1
         inter.broker.protocol.version: "3.1"
       storage:
         type: ephemeral
     zookeeper:
       replicas: 1
       storage:
         type: ephemeral
     entityOperator:
       topicOperator: {}
       userOperator: {}
   ---
   apiVersion: kafka.strimzi.io/v1beta2
   kind: KafkaTopic
   metadata:
     name: events-sample
     namespace: default
     labels:
       strimzi.io/cluster: kafka-server
   spec:
     partitions: 10
     replicas: 1
     config:
       retention.ms: 7200000
       segment.bytes: 1073741824
   ```

3. 运行以下命令在默认命名空间中部署一个名为 `kafka-server` 的 1-replica Kafka 服务器和一个名为 `events-sample` 的 1-replica Kafka 主题。此命令创建的 Kafka 和 Zookeeper 集群的存储类型为 **ephemeral**，并使用 emptyDir 进行演示。

   ```shell
   kubectl apply -f kafka.yaml
   ```

4. 运行以下命令检查 pod 状态，并等待 Kafka 和 Zookeeper 启动并运行。

   ```shell
   $ kubectl get po
   NAME                                              READY   STATUS        RESTARTS   AGE
   kafka-server-entity-operator-568957ff84-nmtlw     3/3     Running       0          8m42s
   kafka-server-kafka-0                              1/1     Running       0          9m13s
   kafka-server-zookeeper-0                          1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm         1/1     Running       0          11m
   ```

5. 运行以下命令查看 Kafka 集群的元数据。

   ```shell
   kafkacat -L -b kafka-server-kafka-brokers:9092
   ```

## 触发同步函数

### 创建 EventSource

1. 使用以下内容创建一个 EventSource 配置文件（例如，`eventsource-sink.yaml`）。

   {{% alert title="注意" color="success" %}}

   - 以下示例定义了一个名为 `my-eventsource` 的事件源，并将指定 Kafka 服务器生成的事件标记为 `sample-one` 事件。
   - `spec.sink` 引用了在先决条件中创建的目标函数（Knative 服务）。

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: my-eventsource
   spec:
     logLevel: "2"
     kafka:
       sample-one:
         brokers: "kafka-server-kafka-brokers.default.svc.cluster.local:9092"
         topic: "events-sample"
         authRequired: false
     sink:
       uri: "http://openfunction.io.svc.cluster.local/default/sink"
   ```

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f eventsource-sink.yaml
   ```

3. 运行以下命令检查结果。

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME             EVENTBUS   SINK   STATUS
   my-eventsource                     Ready

   $ kubectl get components
   NAME                                                      AGE
   serving-8f6md-component-esc-kafka-sample-one-r527t        68m
   serving-8f6md-component-ts-my-eventsource-default-wz8jt   68m

   $ kubectl get deployments.apps
   NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
   serving-8f6md-deployment-v100-pg9sd            1/1     1            1           68m
   ```

   {{% alert title="注意" color="success" %}}

   在此示例中触发同步函数，EventSource 控制器的工作流程描述如下：

   1. 创建一个名为 `my-eventsource` 的 EventSource 自定义资源。
   2. 创建一个名为 `serving-xxxxx-component-esc-kafka-sample-one-xxxxx` 的 Dapr 组件，使 EventSource 能够与事件源关联。
   3. 创建一个名为 `serving-xxxxx-component-ts-my-eventsource-default-xxxxx` 的 Dapr 组件，使 EventSource 能够与 sink 函数关联。
   4. 创建一个名为 `serving-xxxxx-deployment-v100-xxxxx-xxxxxxxxxx-xxxxx` 的 Deployment，用于处理事件。

   {{% /alert %}}

### 创建事件生产者

要启动目标函数，需要创建一些事件来触发函数。

1. 使用以下内容创建一个事件生产者配置文件（例如，`events-producer.yaml`）。

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
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
       runtime: "async"
       inputs:
         - name: cron
           component: cron
       outputs:
         - name: target
           component: kafka-server
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

3. 运行以下命令实时检查结果。

   ```shell
   $ kubectl get po --watch
   NAME                                                           READY   STATUS              RESTARTS   AGE
   serving-k6zw8-deployment-v100-fbtdc-dc96c4589-s25dh            0/2     ContainerCreating   0          1s
   serving-8f6md-deployment-v100-pg9sd-6666c5577f-4rpdg           2/2     Running             0          23m
   serving-k6zw8-deployment-v100-fbtdc-dc96c4589-s25dh            0/2     ContainerCreating   0          1s
   serving-k6zw8-deployment-v100-fbtdc-dc96c4589-s25dh            1/2     Running             0          5s
   serving-k6zw8-deployment-v100-fbtdc-dc96c4589-s25dh            2/2     Running             0          8s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      0/2     Pending             0          0s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      0/2     Pending             0          0s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      0/2     ContainerCreating   0          0s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      0/2     ContainerCreating   0          2s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      1/2     Running             0          4s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      1/2     Running             0          4s
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-8n6mk      2/2     Running             0          4s
   ```