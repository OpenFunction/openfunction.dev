---
title: "Use EventBus and Trigger"
linkTitle: "Use EventBus and Trigger"
weight: 4520
description:
---

This document gives an example of how to use EventBus and Trigger.

## Prerequisites

- You need to create a function as the target function to be triggered. Please refer to [Create a function](../event-source#create-a-function) for more details.
- You need to create a Kafka cluster. Please refer to [Create a Kafka cluster](../event-source#create-a-kafka-cluster) for more details.

## Deploy an NATS streaming server

Run the following commands to deploy an NATS streaming server. This document uses `nats://nats.default:4222` as the access address of the NATS streaming server and `stan` as the cluster ID. For more information, see [NATS Streaming (STAN)](https://github.com/nats-io/k8s/tree/main/helm/charts/stan#tldr).

```shell
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
helm install nats nats/nats
helm install stan nats/stan --set stan.nats.url=nats://nats:4222
```

## Create an OpenFuncAsync Runtime Function

1. Use the following content to create a configuration file (for example, `openfuncasync-function.yaml`) for the target function, which is triggered by the Trigger CRD and prints the received message.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: trigger-target
   spec:
     version: "v1.0.0"
     image: openfunctiondev/v1beta1-trigger-target:latest
     port: 8080
     serving:
       runtime: "async"
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
       inputs:
         - name: autoscaling-pubsub
           component: eventbus
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
   
2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f openfuncasync-function.yaml
   ```

## Create an EventBus and an EventSource

1. Use the following content to create a configuration file (for example, `eventbus.yaml`) for an EventBus.

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

2. Use the following content to create a configuration file (for example, `eventsource.yaml`) for an EventSource.

   {{% alert title="Note" color="success" %}}

   Set the name of the event bus through `spec.eventBus`.

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

3. Run the following commands to apply these configuration files.

   ```shell
   kubectl apply -f eventbus.yaml
   kubectl apply -f eventsource.yaml
   ```

4. Run the following commands to check the results.

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME             EVENTBUS   SINK   STATUS
   my-eventsource   default           Ready
   
   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   6m53s
   
   $ kubectl get components
   NAME                                                 AGE
   serving-6r5dl-component-eventbus-jlpqf               11m
   serving-9689d-component-ebfes-my-eventsource-cmcbw   6m57s
   serving-9689d-component-esc-kafka-sample-two-l99cg   6m57s
   serving-k6zw8-component-cron-9x8hl                   61m
   serving-k6zw8-component-kafka-server-sjrzs           61m
   
   $ kubectl get deployments.apps
   NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
   serving-6r5dl-deployment-v100-m7nq2        0/0     0            0           12m
   serving-9689d-deployment-v100-5qdvk        1/1     1            1           7m17s
   ```

   {{% alert title="Note" color="success" %}}
   
   In the case of using the event bus, the workflow of the EventSource controller is described as follows:
   
   1. Create an EventSource custom resource named `my-eventsource`.
   2. Retrieve and reorganize the configuration of the EventBus, including the EventBus name (`default` in this example) and the name of the Dapr component associated with the EventBus.
   3. Create a Dapr component named `serving-xxxxx-component-ebfes-my-eventsource-xxxxx` to enable the EventSource to associate with the event bus.
   4. Create a Dapr component named `serving-xxxxx-component-esc-kafka-sample-two-xxxxx` to enable the EventSource to associate with the event source.
   5. Create a Deployment named `serving-xxxxx-deployment-v100-xxxxx` for processing events.
   
   {{% /alert %}}

## Create a Trigger

1. Use the following content to create a configuration file (for example, `trigger.yaml`) for a Trigger.

   {{% alert title="Note" color="success" %}}

   - Set the event bus associated with the Trigger through `spec.eventBus`.
   - Set the event input source through `spec.inputs`.
   - This is a simple trigger that collects events from the EventBus named `default`. When it retrieves a `sample-two` event from the EventSource `my-eventsource`, it triggers a Knative service named `function-sample-serving-qrdx8-ksvc-fwml8` and sends the event to the topic `metrics` of the event bus at the same time. 

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
   
2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f trigger.yaml
   ```

3. Run the following commands to check the results.

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
   
   {{% alert title="Note" color="success" %}}
   
   In the case of using the event bus, the workflow of the Trigger controller is as follows:
   
   1. Create a Trigger custom resource named `my-trigger`.
   2. Retrieve and reorganize the configuration of the EventBus, including the EventBus name (`default` in this example) and the name of the Dapr component associated with the EventBus.
   3. Create a Dapr component named `serving-xxxxx-component-ebft-my-trigger-xxxxx` to enable the Trigger to associatie with the event bus.
   4. Create a Deployment named `serving-xxxxx-deployment-v100-xxxxx` for processing trigger tasks.
   
   {{% /alert %}}

## Create an Event Producer

1. Use the following content to create an event producer configuration file (for example, `events-producer.yaml`).

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

2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f events-producer.yaml
   ```
   
3. Run the following commands to observe changes of the target asynchronous function.

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



