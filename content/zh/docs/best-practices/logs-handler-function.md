---
title: "Elastic 日志告警"
linkTitle: "Elastic 日志告警"
weight: 5400
description: >
    学习如何创建一个异步函数来查找错误日志。
---

本文档描述了如何创建一个异步函数来查找错误日志。

## 概述

本文档使用一个异步函数来分析 Kafka 中的日志流，以找出错误日志。异步函数将然后向 Slack 发送告警。以下图表说明了整个工作流程。

![](/images/docs/en/best-practices/logs-handler-function/elastic-log-processing.png)

## 先决条件

- 您已经[安装了 OpenFunction](../../getting-started/installation/)。
- 您已经[创建了一个 secret](../../getting-started/quickstarts/prerequisites/)。

## 创建 Kafka 服务器和主题

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
     name: kafka-logs-receiver
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
     name: logs
     namespace: default
     labels:
       strimzi.io/cluster: kafka-logs-receiver
   spec:
     partitions: 10
     replicas: 1
     config:
       retention.ms: 7200000
       segment.bytes: 1073741824
   ```

3. 运行以下命令在默认命名空间中部署一个名为 `kafka-logs-receiver` 的 1 副本 Kafka 服务器和一个名为 `logs` 的 1 副本 Kafka 主题。

   ```shell
   kubectl apply -f kafka.yaml
   ```

4. 运行以下命令检查 pod 状态，等待 Kafka 和 Zookeeper 启动并运行。

   ```shell
   $ kubectl get po
   NAME                                                     READY   STATUS        RESTARTS   AGE
   kafka-logs-receiver-entity-operator-57dc457ccc-tlqqs     3/3     Running       0          8m42s
   kafka-logs-receiver-kafka-0                              1/1     Running       0          9m13s
   kafka-logs-receiver-zookeeper-0                          1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm                1/1     Running       0          11m
   ```

5. 运行以下命令查看 Kafka 集群的元数据。

   ```shell
   # 启动一个实用工具 pod。
   $ kubectl run utils --image=arunvelsriram/utils -i --tty --rm
   # 检查 Kafka 集群的元数据。
   $ kafkacat -L -b kafka-logs-receiver-kafka-brokers:9092
   ```

## 创建日志处理函数

1. 使用以下示例 YAML 文件创建一个清单 `logs-handler-function.yaml`，并修改 `spec.image` 的值以设置您自己的镜像仓库地址。

```yaml
   apiVersion: core.openfunction.io/v1beta2
   kind: Function
   metadata:
     name: logs-async-handler
     namespace: default
   spec:
     build:
       builder: openfunction/builder-go:latest
       env:
         FUNC_CLEAR_SOURCE: "true"
         FUNC_NAME: LogsHandler
       srcRepo:
         revision: main
         sourceSubPath: functions/async/logs-handler-function/
         url: https://github.com/OpenFunction/samples.git
     image: openfunctiondev/logs-async-handler:v1
     imageCredentials:
       name: push-secret
     serving:
       bindings:
         kafka-receiver:
           metadata:
             - name: brokers
               value: kafka-server-kafka-brokers:9092
             - name: authRequired
               value: "false"
             - name: publishTopic
               value: logs
             - name: topics
               value: logs
             - name: consumerGroup
               value: logs-handler
           type: bindings.kafka
           version: v1
         notification-manager:
           metadata:
             - name: url
               value: http://notification-manager-svc.kubesphere-monitoring-system.svc.cluster.local:19093/api/v2/alerts
           type: bindings.http
           version: v1
       outputs:
         - dapr:
             name: notification-manager
             operation: post
             type: bindings.http
       scaleOptions:
         keda:
           scaledObject:
             advanced:
               horizontalPodAutoscalerConfig:
                 behavior:
                   scaleDown:
                     policies:
                       - periodSeconds: 15
                         type: Percent
                         value: 50
                     stabilizationWindowSeconds: 45
                   scaleUp:
                     stabilizationWindowSeconds: 0
             cooldownPeriod: 60
             pollingInterval: 15
           triggers:
             - metadata:
                 bootstrapServers: kafka-server-kafka-brokers.default.svc.cluster.local:9092
                 consumerGroup: logs-handler
                 lagThreshold: "20"
                 topic: logs
               type: kafka
         maxReplicas: 10
         minReplicas: 0
       template:
         containers:
           - imagePullPolicy: IfNotPresent
             name: function
       triggers:
         dapr:
           - name: kafka-receiver
             type: bindings.kafka
       workloadType: Deployment
     version: v2.0.0
     workloadRuntime: OCIContainer
```

2. 运行以下命令创建函数 `logs-async-handler`。

   ```shell
   kubectl apply -f logs-handler-function.yaml
   ```

3. 日志处理函数将由 Kafka 中 logs 主题的消息触发。
