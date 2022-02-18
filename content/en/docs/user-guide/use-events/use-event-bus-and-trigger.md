---
title: "Use EventBus and Trigger"
linkTitle: "Use EventBus and Trigger"
weight: 4520
description: >	
    Learn how to use EventBus and Trigger.
---

This document gives an example of how to use EventBus and Trigger.

## Prerequisites

- You need to prepare a Knative runtime function as a target function. For more information about how to create a Knative runtime function, see [Your First Function: Go](../../../getting-started/your-first-function/function-go).

  {{% alert title="Note" color="success" %}}

  This document uses `function-sample-serving-ksvc` as an example of the name of the target function (Knative service). To obtain the name of your function, run the `kubectl get ksvc` command.

  {{% /alert %}}

- You need to prepare a Kafka server as the event source. For more information, see [Setting up a Kafka in Kubernetes](https://github.com/dapr/quickstarts/tree/master/bindings#setting-up-a-kafka-in-kubernetes).

  {{% alert title="Note" color="success" %}}

  This document uses `dapr-kafka.kafka:9092` as an example of the access address of the Kafka server.

  {{% /alert %}}
  
- You need to prepare a NATS streaming server as the event bus. For more information, see [Running NATS on Kubernetes](https://nats-io.github.io/k8s/).

  {{% alert title="Note" color="success" %}}

  This document uses `nats://nats.default:4222` as an example of the access address of the NATS streaming server and `stan` as an example of the cluster ID.

  {{% /alert %}}

## Create an OpenFuncAsync Runtime Function

1. Use the following content to create a configuration file (for example, `openfuncasync-function.yaml`) for a function, which serves to print the received message.

   {{% alert title="Note" color="success" %}}

   Make sure you replace the `<your Docker Hub ID>` in `spec.image` with your Docker Hub ID.

   {{% /alert %}}

   ```yaml
   apiVersion: core.openfunction.io/v1alpha1
   kind: Function
   metadata:
     name: autoscaling-subscriber
   spec:
     version: "v1.0.0"
     image: <your Docker Hub ID>/autoscaling-subscriber:latest
     imageCredentials:
       name: push-secret
     build:
       builder: openfunctiondev/go115-builder:v0.2.0
       env:
         FUNC_NAME: "Subscriber"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/OpenFuncAsync/pubsub/subscriber"
     serving:
       runtime: "OpenFuncAsync"
       openFuncAsync:
         dapr:
           inputs:
             - name: autoscaling-pubsub
               type: pubsub
               topic: metrics
           annotations:
             dapr.io/log-level: "debug"
           components:
             - name: autoscaling-pubsub
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
         keda:
           scaledObject:
             pollingInterval: 15
             minReplicaCount: 0
             maxReplicaCount: 10
             cooldownPeriod: 30
             triggers:
               - type: stan
                 metadata:
                   natsServerMonitoringEndpoint: "stan-0.stan.default.svc.cluster.local:8222"
                   queueGroup: "grp1"
                   durableName: "ImDurable"
                   subject: "metrics"
                   lagThreshold: "10"
   ```

2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f openfuncasync-function.yaml
   ```

## Create an EventBus and an EventSource

1. Use the following content to create a configuration file (for example, `eventbus-default.yaml`) for an EventBus.

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventBus
   metadata:
     name: default
   spec:
     natsStreaming:
       natsURL: "nats://nats.default:4222"
       natsStreamingClusterID: "stan"
       subscriptionType: queue
   ```

2. Use the following content to create a configuration file (for example, `eventsource-eventbus.yaml`) for an EventSource.

   {{% alert title="Note" color="success" %}}

   Set the name of the event bus through `spec.eventBus`.

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: my-eventsource
   spec:
     eventBus: "default"
     kafka:
       sample-two:
         brokers: dapr-kafka.kafka:9092
         topic: sample
         authRequired: false
   ```

3. Run the following commands to apply these configuration files.

   ```shell
   kubectl apply -f eventbus-default.yaml
   kubectl apply -f eventsource-eventbus.yaml
   ```

4. Run the following commands to check the results.

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME             EVENTBUS   SINK
   my-eventsource   default
   
   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   10m
   
   $ kubectl get components
   NAME                                          AGE
   eventsource-eventbus-my-eventsource           28s
   eventsource-my-eventsource-kafka-sample-two   28s
   
   $ kubectl get deployments.apps
   NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
   eventsource-my-eventsource-kafka-sample-two    1/1     1            1           4m53s
   ```

   {{% alert title="Note" color="success" %}}

   In the case of using the event bus, the workflow of the EventSource controller is as follows:

   1. Create an EventSource custom resource named `my-eventsource`.
   2. Retrieve and reorganize the configuration of the EventBus, including the EventBus name (`default` in this example) and the name of the Dapr component associated with the EventBus (`eventsource-eventbus-my-eventsource` in this example).
   3. Create a Dapr component named `eventsource-eventbus-my-eventsource` for associating the event bus.
   4. Create a Dapr component named `eventsource-my-eventsource-kafka-sample-two` for associating the event source.
   5. Create a Deployment named `eventsource-my-eventsource-kafka-sample-two` for processing events.

   {{% /alert %}}

## Create a Trigger

1. Use the following content to create a configuration file (for example, `trigger.yaml`) for a Trigger.

   {{% alert title="Note" color="success" %}}

   - Set the event bus associated with the Trigger through `spec.eventBus`.
   - Set the event input source through `spec.inputs`.
   - This is a simple trigger that collects events from the EventBus `default`. When it retrieves a `sample-two` event from the EventSource `my-eventsource`, it triggers a Knative service named `function-sample-serving-qrdx8-ksvc-fwml8` and sends the event to the topic `metrics` of the event bus at the same time. 

   {{% /alert %}}

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: Trigger
   metadata:
     name: my-trigger
   spec:
     eventBus: "default"
     inputs:
       inputDemo:
         eventSource: "my-eventsource"
         event: "sample-two"
     subscribers:
     - condition: inputDemo
       sink:
         ref:
           apiVersion: serving.knative.dev/v1
           kind: Service
           name: function-sample-serving-qrdx8-ksvc-fwml8
           namespace: default
       topic: "metrics"
   ```

2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f trigger.yaml
   ```

3. Run the following commands to check the results.

   ```shell
   $ kubectl get triggers.events.openfunction.io
   NAME         AGE
   my-trigger   34m
   
   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   62m
   
   $ kubectl get components
   autoscaling-pubsub                                                         90m
   eventbus-eventsource-my-eventsource                                        161m
   eventbus-trigger-my-trigger                                                161m
   eventsource-my-eventsource-kafka-sample-two                                161m
   trigger-sink-my-trigger-default-function-sample-serving-qrdx8-ksvc-fwml8   161m
   
   $ kubectl get deployments.apps
   NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
   trigger-my-trigger      1/1     1            1           2m52s
   ```
   
   {{% alert title="Note" color="success" %}}
   
   In the case of using the event bus, the workflow of the Trigger controller is as follows:
   
   1. Create a Trigger custom resource named `my-trigger`.
   2. Retrieve and reorganize the configuration of the EventBus, including the EventBus name (`default` in this example) and the name of the Dapr component associated with the EventBus (`eventbus-trigger-my-trigger` in this example).
   3. Create a Dapr component named `eventbus-trigger-my-trigger` for associating the event bus.
   4. Create a Dapr component named `trigger-sink-my-trigger-default-function-sample-serving-qrdx8-ksvc-fwml8` for associating the target Knative function.
   5. Create a Deployment named `trigger-my-trigger` for processing trigger tasks.
   
   {{% /alert %}}

## Create an Event Producer

1. Use the following content to create an event producer configuration file (for example, `events-producer.yaml`).

   {{% alert title="Note" color="success" %}}

   - This document uses the image `openfunctiondev/events-producer:latest` as an example image, which publishes events to the event source at a rate of one event per 5 seconds.
   - To use this image as an event producer, you need to use an environment variable to set the value of `TARGET_NAME` to the name of the Dapr component for associating the event source, namely `eventsource-my-eventsource-kafka-sample-two`.

   {{% /alert %}}

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: events-producer
     labels:
       app: eventsproducer
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: eventsproducer
     template:
       metadata:
         labels:
           app: eventsproducer
         annotations:
           dapr.io/enabled: "true"
           dapr.io/app-id: "events-producer"
           dapr.io/log-as-json: "true"
       spec:
         containers:
           - name: producer
             image: openfunctiondev/events-producer:latest
             imagePullPolicy: Always
             env:
               - name: TARGET_NAME
                 value: "eventsource-my-eventsource-kafka-sample-two"
   ```

2. Run the following command to apply the configuration file.

   ```shell
   kubectl apply -f events-producer.yaml
   ```

3. Run the following command to check the results in real time.

   ```shell
   $ kubectl get po --watch
   NAME                                                           READY   STATUS              RESTARTS   AGE
   events-producer-86679d99fb-4tlbt                               0/2     ContainerCreating   0          2s
   eventsource-my-eventsource-kafka-sample-two-7695c6cfdd-j2g5b   2/2     Running             0          58m
   trigger-my-trigger-7b799c7f7d-4ph77                            2/2     Running             0          37m
   events-producer-86679d99fb-4tlbt                               0/2     ContainerCreating   0          2s
   events-producer-86679d99fb-4tlbt                               1/2     Running             0          6s
   events-producer-86679d99fb-4tlbt                               2/2     Running             0          10s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   0/2     Pending             0          0s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   0/2     Pending             0          0s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   0/2     ContainerCreating   0          0s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   0/2     ContainerCreating   0          2s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   1/2     Running             0          3s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   1/2     Running             0          4s
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-8cpxsj   2/2     Running             0          4s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   0/2     Pending             0          0s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   0/2     Pending             0          0s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   0/2     ContainerCreating   0          0s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   0/2     ContainerCreating   0          1s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   1/2     Running             0          3s
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88d8bf6m   2/2     Running             0          11s
   ```



