---
title: "使用 ClusterEventBus"
linkTitle: "使用 ClusterEventBus"
weight: 4540
description:
---

本文档描述了如何使用 ClusterEventBus。

## 先决条件

您已完成 [使用 EventBus 和 Trigger](../event-bus-and-trigger) 中描述的步骤。

## 使用 ClusterEventBus

1. 使用以下内容创建一个 ClusterEventBus 配置文件（例如，`clustereventbus.yaml`）。

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: ClusterEventBus
   metadata:
     name: default
   spec:
     natsStreaming:
       natsURL: "nats://nats.default:4222"
       natsStreamingClusterID: "stan"
       subscriptionType: "queue"
       durableSubscriptionName: "ImDurable"
   ```

2. 运行以下命令删除 EventBus。

   ```shell
   kubectl delete eventbus.events.openfunction.io default
   ```

3. 运行以下命令应用配置文件。

   ```shell
   kubectl apply -f clustereventbus.yaml
   ```

4. 运行以下命令检查结果。

   ```shell
   $ kubectl get eventbus.events.openfunction.io
   No resources found in default namespace.

   $ kubectl get clustereventbus.events.openfunction.io
   NAME      AGE
   default   21s
   ```