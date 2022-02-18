---
title: "Use ClusterEventBus"
linkTitle: "Use ClusterEventBus"
weight: 4540
description: >	
    Learn how to use a ClusterEventBus.
---

This document describes how to use a ClusterEventBus.

## Prerequisites

You need to be familiar with the example given in [Use EventBus and Trigger](../use-event-bus-and-trigger).

## Use a ClusterEventBus

1. Use the following content to create a ClusterEventBus configuration file (for example, `clustereventbus-default.yaml`).

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: ClusterEventBus
   metadata:
     name: default
   spec:
     natsStreaming:
       natsURL: "nats://nats.default:4222"
       natsStreamingClusterID: "stan"
       subscriptionType: queue
   ```
   
2. Run the following command to delete EventBus.

   ```shell
   kubectl delete eventbus.events.openfunction.io default
   ```

2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f clustereventbus-default.yaml
   ```

3. Run the following commands to check the results.

   ```shell
   $ kubectl get eventbus.events.openfunction.io
   No resources found in default namespace.
   
   $ kubectl get clustereventbus.events.openfunction.io
   NAME      AGE
   default   21s
   ```
   
   {{% alert title="Note" color="success" %}}
   
   If there are no other changes, you can see that the event bus is still working properly in the whole sample.
   
   {{% /alert %}}

