---
title: "创建一个基于 Knative 的函数以通过 Dapr 组件与中间件交互"
linkTitle: "创建一个基于 Knative 的函数以通过 Dapr 组件与中间件交互"
weight: 5200
description: >
    了解如何创建一个基于 Knative 的函数，通过 Dapr 组件与中间件交互。
---

本文档描述了如何创建一个基于 Knative 的函数，通过 Dapr 组件与中间件交互。

## 概述

与异步函数类似，基于 Knative 运行时的函数可以通过 Dapr 组件与中间件交互。本文档使用两个函数，`function-front` 和 `kafka-input`，进行演示。

以下图表说明了这些函数之间的关系。

![](/images/docs/en/best-practices/interact-with-dapr-output-binding/knative-dapr.png)

## 先决条件

- 您已经[安装了 OpenFunction](../../getting-started/installation/)。
- 您已经[创建了一个 secret](../../getting-started/quickstarts/prerequisites/)。

## 创建 Kafka 服务器和主题

1. 运行以下命令，在默认命名空间中安装 [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator)。

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
     name: sample-topic
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

3. 运行以下命令，在默认命名空间中部署一个名为 `kafka-server` 的 1 副本 Kafka 服务器和一个名为 `sample-topic` 的 1 副本 Kafka 主题。

   ```shell
   kubectl apply -f kafka.yaml
   ```

4. 运行以下命令，检查 pod 状态，等待 Kafka 和 Zookeeper 启动并运行。

   ```shell
   $ kubectl get po
   NAME                                              READY   STATUS        RESTARTS   AGE
   kafka-server-entity-operator-568957ff84-nmtlw     3/3     Running       0          8m42s
   kafka-server-kafka-0                              1/1     Running       0          9m13s
   kafka-server-zookeeper-0                          1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm         1/1     Running       0          11m
   ```

5. 运行以下命令，查看 Kafka 集群的元数据。

   ```shell
   # 启动一个实用工具 pod。
   $ kubectl run utils --image=arunvelsriram/utils -i --tty --rm
   # 检查 Kafka 集群的元数据。
   $ kafkacat -L -b kafka-server-kafka-brokers:9092
   ```

## 创建函数

1. 使用以下示例 YAML 文件创建一个清单 `kafka-input.yaml`，并修改 `spec.image` 的值以设置您自己的镜像仓库地址。字段 `spec.serving.inputs` 定义了一个指向 Kafka 服务器的 Dapr 组件的输入源。这意味着 `kafka-input` 函数将由 Kafka 服务器的主题 `sample-topic` 中的事件驱动。

   ```yaml
   apiVersion: core.openfunction.io/v1beta2
   kind: Function
   metadata:
     name: kafka-input
   spec:
     version: "v1.0.0"
     image: <your registry name>/kafka-input:latest
     imageCredentials:
       name: push-secret
     build:
       builder: openfunction/builder-go:latest
       env:
         FUNC_NAME: "HandleKafkaInput"
         FUNC_CLEAR_SOURCE: "true"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/async/bindings/kafka-input"
         revision: "main"
     serving:
       scaleOptions:
         minReplicas: 0
         maxReplicas: 10
         keda:
           triggers:
           - type: kafka
             metadata:
               topic: sample-topic
               bootstrapServers: kafka-server-kafka-brokers.default.svc:9092
               consumerGroup: kafka-input
               lagThreshold: "20"
           scaledObject:
             pollingInterval: 15
             cooldownPeriod: 60
             advanced:
               horizontalPodAutoscalerConfig:
                 behavior:
                   scaleDown:
                     stabilizationWindowSeconds: 45
                     policies:
                     - type: Percent
                       value: 50
                       periodSeconds: 15
                   scaleUp:
                     stabilizationWindowSeconds: 0

       triggers:
         dapr:
         - name: target-topic
           type: bindings.kafka
       bindings:
         target-topic:
           type: bindings.kafka
           version: v1
           metadata:
             - name: brokers
               value: "kafka-server-kafka-brokers:9092"
             - name: topics
               value: "sample-topic"
             - name: consumerGroup
               value: "kafka-input"
             - name: publishTopic
               value: "sample-topic"
             - name: authRequired
               value: "false"
       template:
         containers:
           - name: function
             imagePullPolicy: Always
   ```

2. 运行以下命令创建函数 `kafka-input`。

   ```shell
   kubectl apply -f kafka-input.yaml
   ```

3. 使用以下示例 YAML 文件创建一个清单 `function-front.yaml`，并修改 `spec.image` 的值以设置您自己的镜像仓库地址。

   ```yaml
      apiVersion: core.openfunction.io/v1beta2
      kind: Function
      metadata:
        name: function-front
      spec:
        version: "v1.0.0"
        image: "<your registry name>/sample-knative-dapr:latest"
        imageCredentials:
          name: push-secret
        build:
          builder: openfunction/builder-go:latest
          env:
            FUNC_NAME: "ForwardToKafka"
            FUNC_CLEAR_SOURCE: "true"
          srcRepo:
            url: "https://github.com/OpenFunction/samples.git"
            sourceSubPath: "functions/knative/with-output-binding"
            revision: "main"
        serving:
           hooks:
             pre:
               - plugin-custom
               - plugin-example
             post:
               - plugin-example
               - plugin-custom
          scaleOptions:
            minReplicas: 0
            maxReplicas: 5
          outputs:
            - dapr:
                name: kafka-server
                operation: "create"
          bindings:
            kafka-server:
              type: bindings.kafka
              version: v1
              metadata:
                - name: brokers
                  value: "kafka-server-kafka-brokers:9092"
                - name: authRequired
                  value: "false"
                - name: publishTopic
                  value: "sample-topic"
                - name: topics
                  value: "sample-topic"
                - name: consumerGroup
                  value: "function-front"
          template:
            containers:
              - name: function
                imagePullPolicy: Always
    ```


   {{% alert title="注意" color="success" %}}

   `metadata.plugins.pre` 定义了在执行用户函数之前需要调用的插件的顺序。`metadata.plugins.post` 定义了在执行用户函数之后需要调用的插件的顺序。有关这两个插件的逻辑以及执行插件后的效果的更多信息，请参阅 [插件机制](https://github.com/OpenFunction/samples/blob/main/functions-framework/README.md#plugin-mechanism)。

   {{% /alert %}}

4. 在清单中，`spec.serving.outputs` 定义了一个指向 Kafka 服务器的 Dapr 组件的输出。这使您可以在 `function-front` 函数中向输出 `target` 发送自定义内容。

   ```go
   func Sender(ctx ofctx.Context, in []byte) (ofctx.Out, error) {
     ...
    _, err := ctx.Send("target", greeting)
    ...
   }
   ```

5. 运行以下命令创建函数 `function-front`。

   ```shell
   kubectl apply -f function-front.yaml
   ```

## 检查结果

1. 运行以下命令查看函数的状态。

   ```shell
   $ kubectl get functions.core.openfunction.io

   NAME             BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         URL                                             AGE
   function-front   Succeeded    Running        builder-bhbtk   serving-vc6jw   https://openfunction.io/default/function-front   2m41s
   kafka-input      Succeeded    Running        builder-dprfd   serving-75vrt                                                   2m21s
   ```

   {{% alert title="注意" color="success" %}}

   `URL`，由 OpenFunction Domain 提供，是可以访问的地址。要通过此 URL 地址访问函数，您需要确保 DNS 可以解析此地址。

   {{% /alert %}}

2. 运行以下命令在集群中创建一个用于访问函数的 pod。

   ```shell
   kubectl run curl --image=radial/busyboxplus:curl -i --tty --rm
   ```

3. 运行以下命令通过 `URL` 访问函数。

   ```shell
   [ root@curl:/ ]$ curl -d '{"message":"Awesome OpenFunction!"}' -H "Content-Type: application/json" -X POST http://openfunction.io.svc.cluster.local/default/function-front
   ```

4. 运行以下命令查看 `function-front` 的日志。

   ```shell
   kubectl logs -f \
     $(kubectl get po -l \
     openfunction.io/serving=$(kubectl get functions function-front -o jsonpath='{.status.serving.resourceRef}') \
     -o jsonpath='{.items[0].metadata.name}') \
     function
   ```

   输出如下所示。

   ```shell
   dapr client initializing for: 127.0.0.1:50001
   I0125 06:51:55.584973       1 framework.go:107] Plugins for pre-hook stage:
   I0125 06:51:55.585044       1 framework.go:110] - plugin-custom
   I0125 06:51:55.585052       1 framework.go:110] - plugin-example
   I0125 06:51:55.585057       1 framework.go:115] Plugins for post-hook stage:
   I0125 06:51:55.585062       1 framework.go:118] - plugin-custom
   I0125 06:51:55.585067       1 framework.go:118] - plugin-example
   I0125 06:51:55.585179       1 knative.go:46] Knative Function serving http: listening on port 8080
   2022/01/25 06:52:02 http - Data: {"message":"Awesome OpenFunction!"}
   I0125 06:52:02.246450       1 plugin-example.go:83] the sum is: 2
   ```

5. 运行以下命令查看 `kafka-input` 的日志。

   ```shell
   kubectl logs -f \
     $(kubectl get po -l \
     openfunction.io/serving=$(kubectl get functions kafka-input -o jsonpath='{.status.serving.resourceRef}') \
     -o jsonpath='{.items[0].metadata.name}') \
     function
   ```

   输出如下所示。

   ```shell
   dapr client initializing for: 127.0.0.1:50001
   I0125 06:35:28.332381       1 framework.go:107] Plugins for pre-hook stage:
   I0125 06:35:28.332863       1 framework.go:115] Plugins for post-hook stage:
   I0125 06:35:28.333749       1 async.go:39] Async Function serving grpc: listening on port 8080
   message from Kafka '{Awesome OpenFunction!}'
   ```
