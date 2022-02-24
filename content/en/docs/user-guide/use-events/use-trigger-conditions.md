---
title: "Use Trigger Conditions"
linkTitle: "Use Trigger Conditions"
weight: 4550
description: >	
    Learn how to use Trigger conditions.
---

This document describes how to use Trigger conditions.

## Prerequisites

You need to be familiar with the example given in [Use EventBus and Trigger](../use-event-bus-and-trigger).

## Use Trigger Conditions

1. Use the following content to create an EventSource configuration file (for example, `eventsource-a.yaml`).

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: eventsource-a
   spec:
     eventBus: "default"
     kafka:
       sample-five:
         brokers: dapr-kafka.kafka:9092
         topic: sample
         authRequired: false
   ```
   
1. Use the following content to create another EventSource configuration file (for example, `eventsource-a.yaml`).

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: EventSource
   metadata:
     name: eventsource-b
   spec:
     eventBus: "default"
     cron:
       sample-five:
         schedule: "@every 5s"
   ```
   
3. Use the following content to create a configuration file (for example, `condition-trigger.yaml`) for a Trigger with `condition`.

   ```yaml
   apiVersion: events.openfunction.io/v1alpha1
   kind: Trigger
   metadata:
     name: condition-trigger
   spec:
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
         ref:
           apiVersion: serving.knative.dev/v1
           kind: Service
           name: function-sample-serving-qrdx8-ksvc-fwml8
           namespace: default
     - condition: eventA && eventB
       topic: "metrics"
   ```

   {{% alert title="Note" color="success" %}}

   In this example, two input sources and two subscribers are defined, and their triggering relationship is as follows:

   - When input `eventB` is received, the input event is sent to the Knative service.
   - When input `eventB` and input `eventA` are received, the input event is sent to the metrics topic of the event bus. (The above step is also effective)

   {{% /alert %}}

4. Run the following commands to apply the configuration files.

   ```shell
   kubectl apply -f eventsource-a.yaml
   kubectl apply -f eventsource-b.yaml
   kubectl apply -f condition-trigger.yaml
   ```

3. Run the following commands to check the results.

   ```shell
   $ kubectl get eventsources.events.openfunction.io
   NAME            EVENTBUS   SINK   STATUS    COMPONENTS   WORKLOADS
   eventsource-a   default           Running   2/2          1/1
   eventsource-b   default           Running   2/2          1/1
   
   $ kubectl get triggers.events.openfunction.io
   NAME                EVENTBUS   STATUS    COMPONENTS   WORKLOADS
   condition-trigger   default    Running   2/2          1/1
   
   $ kubectl get eventbus.events.openfunction.io
   NAME      AGE
   default   12s
   ```

6. Run the following command, and the output shows that the `eventB` condition in the Trigger is matched and the Knative service is triggered as the event source `eventsource-b` is a cron task.

   ```shell
   $ kubectl get po
   NAME                                                              READY   STATUS    RESTARTS   AGE
   function-sample-serving-qrdx8-ksvc-fwml8-v100-deployment-7n4kpd   2/2     Running   0          11m
   ```

7. Create an event producer by referring to [Create an Event Producer](../use-event-bus-and-trigger#create-an-event-producer).

   {{% alert title="Note" color="success" %}}

   Modity the value of `TARGET_NAME` to `eventsource-eventsource-a-kafka-sample-five`.

   {{% /alert %}}

8. Run the following command, and the output shows that the `eventA && eventB` condition in the Trigger is matched and the event is sent to the `metrics` topic of the event bus at the same time. The OpenFuncAsync function is triggered.

   ```shell
   $ kubectl get po
   NAME                                                              READY   STATUS    RESTARTS   AGE
   autoscaling-subscriber-serving-5qzlq-v100-xp9dw-77bc8cc88dts99v   2/2     Running   0          6s 
   ```

