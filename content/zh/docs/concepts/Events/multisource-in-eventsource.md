---
title: "在一个 EventSource 中使用多个源"
linkTitle: "在一个 EventSource 中使用多个源"
weight: 4530
description:
---

本文档描述了如何在一个 EventSource 中使用多个源。

## 先决条件

- 您需要创建一个作为目标函数的函数以被触发。有关更多详细信息，请参见 [创建函数](../event-source#create-a-function)。
- 您需要创建一个 Kafka 集群。有关更多详细信息，请参见 [创建 Kafka 集群](../event-source#create-a-kafka-cluster)。

## 在一个 EventSource 中使用多个源

1. 使用以下内容创建一个 EventSource 配置文件（例如，`eventsource-multi.yaml`）。

   {{% alert title="注意" color="success" %}}

   - 以下示例定义了一个名为 `my-eventsource` 的事件源，并将指定 Kafka 服务器生成的事件标记为 `sample-three` 事件。
   - `spec.sink` 引用了目标函数（Knative 服务）。
   - `spec.cron` 的配置是每5秒触发一次在 `spec.sink` 中定义的函数。

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: my-eventsource
   spec:
     logLevel: "2"
     kafka:
       sample-three:
         brokers: "kafka-server-kafka-brokers.default.svc.cluster.local:9092"
         topic: "events-sample"
         authRequired: false
     cron:
       sample-three:
         schedule: "@every 5s"
     sink:
       uri: "http://openfunction.io.svc.cluster.local/default/sink"
   ```

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f eventsource-multi.yaml
   ```

3. 运行以下命令观察变化。

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME             EVENTBUS   SINK   STATUS
   my-eventsource                     Ready

   $ kubectl get components
   NAME                                                      AGE
   serving-vqfk5-component-esc-cron-sample-three-dzcpv       35s
   serving-vqfk5-component-esc-kafka-sample-one-nr9pq        35s
   serving-vqfk5-component-ts-my-eventsource-default-q6g6m   35s

   $ kubectl get deployments.apps
   NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
   serving-4x5wh-ksvc-wxbf2-v100-deployment   1/1     1            1           3h14m
   serving-vqfk5-deployment-v100-vdmvj        1/1     1            1           48s
   ```