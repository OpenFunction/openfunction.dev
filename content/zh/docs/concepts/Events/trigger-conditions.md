---
title: "使用带有条件的 Trigger"
linkTitle: "使用带有条件的 Trigger"
weight: 4550
description:
---

本文档描述了如何使用带有条件的触发器。

## 先决条件

您已完成 [使用 EventBus 和 Trigger](../event-bus-and-trigger) 中描述的步骤。

## 使用带有条件的触发器

### 创建两个事件源

1. 使用以下内容创建一个 EventSource 配置文件（例如，`eventsource-a.yaml`）。

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: eventsource-a
   spec:
     logLevel: "2"
     eventBus: "default"
     kafka:
       sample-five:
         brokers: "kafka-server-kafka-brokers.default.svc.cluster.local:9092"
         topic: "events-sample"
         authRequired: false
   ```

2. 使用以下内容创建另一个 EventSource 配置文件（例如，`eventsource-b.yaml`）。

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: eventsource-b
   spec:
     logLevel: "2"
     eventBus: "default"
     cron:
       sample-five:
         schedule: "@every 5s"
   ```

3. 运行以下命令应用这两个配置文件。

   ```shell
   kubectl apply -f eventsource-a.yaml
   kubectl apply -f eventsource-b.yaml
   ```

### 创建带有条件的触发器

1. 使用以下内容创建一个带有 `condition` 的触发器配置文件（例如，`condition-trigger.yaml`）。

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: Trigger
   metadata:
     name: condition-trigger
   spec:
     logLevel: "2"
     eventBus: "default"
     inputs:
       eventA:
         eventSource: "eventsource-a"
         event: "sample-five"
       eventB:
         eventSource: "eventsource-b"
         event: "sample-five"
     subscribers:
     - condition: eventB
       sink:
         uri: "http://openfunction.io.svc.cluster.local/default/sink"
     - condition: eventA && eventB
       topic: "metrics"
   ```

   {{% alert title="注意" color="success" %}}

   在这个例子中，定义了两个输入源和两个订阅者，它们的触发关系描述如下：

   - 当接收到输入 `eventB` 时，将输入事件发送到 Knative 服务。
   - 当接收到输入 `eventB` 和输入 `eventA` 时，将输入事件同时发送到事件总线的 `metrics` 主题和 Knative 服务。

   {{% /alert %}}

2. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f condition-trigger.yaml
   ```

3. 运行以下命令检查结果。

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME            EVENTBUS   SINK   STATUS
   eventsource-a   default           Ready
   eventsource-b   default           Ready

   $ kubectl get triggers.events.openfunction.io
   NAME                EVENTBUS   STATUS
   condition-trigger   default    Ready

   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   12s
   ```

4. 运行以下命令，您可以从输出中看到，由于事件源 `eventsource-b` 是一个 cron 任务，所以触发器中的 `eventB` 条件匹配，触发了 Knative 服务。

   ```shell
   $ kubectl get functions.core.openfunction.io
   NAME                                  BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
   sink                                  Skipped      Running                  serving-4x5wh   https://openfunction.io/default/sink   3h25m

   $ kubectl get po
   NAME                                                        READY   STATUS    RESTARTS   AGE
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-k2jdg   2/2     Running   0          46s
   ```

5. 参考 [创建事件生产者](../event-bus-and-trigger#create-an-event-producer) 创建一个事件生产者。

6. 运行以下命令，您可以从输出中看到，触发器中的 `eventA && eventB` 条件匹配，事件同时发送到事件总线的 `metrics` 主题。触发了 OpenFuncAsync 函数。

   ```shell
   $ kubectl get functions.core.openfunction.io
   NAME                                  BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
   trigger-target                        Skipped      Running                  serving-7hghp                                         103s

   $ kubectl get po
   NAME                                                        READY   STATUS    RESTARTS   AGE
   serving-7hghp-deployment-v100-z8wrf-946b4854d-svf55         2/2     Running   0          18s
   ```