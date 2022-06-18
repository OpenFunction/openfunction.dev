---
title: "Use ClusterEventBus"
linkTitle: "Use ClusterEventBus"
weight: 4540
description:
---

This document describes how to use a ClusterEventBus.

## Prerequisites

You have finished the steps described in [Use EventBus and Trigger](../use-event-bus-and-trigger).

## Use a ClusterEventBus

1. Use the following content to create a ClusterEventBus configuration file (for example, `clustereventbus.yaml`).

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
   
2. Run the following command to delete EventBus.

   ```shell
   kubectl delete eventbus.events.openfunction.io default
   ```

3. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f clustereventbus.yaml
   ```

4. Run the following commands to check the results.

   ```shell
   $ kubectl get eventbus.events.openfunction.io
   No resources found in default namespace.
   
   $ kubectl get clustereventbus.events.openfunction.io
   NAME      AGE
   default   21s
   ```
   
   

