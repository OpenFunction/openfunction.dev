---
title: "Use Trigger Conditions"
linkTitle: "Use Trigger Conditions"
weight: 4550
description: 
---

This document describes how to use Trigger conditions.

## Prerequisites

You have finished the steps described in [Use EventBus and Trigger](../event-bus-and-trigger).

## Use Trigger Conditions

### Create two event sources

1. Use the following content to create an EventSource configuration file (for example, `eventsource-a.yaml`).

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
   
2. Use the following content to create another EventSource configuration file (for example, `eventsource-b.yaml`).

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
   
3. Run the following commands to apply these two configuration files.

   ```shell
   kubectl apply -f eventsource-a.yaml
   kubectl apply -f eventsource-b.yaml
   ```

### Create a trigger with condition

1. Use the following content to create a configuration file (for example, `condition-trigger.yaml`) for a Trigger with `condition`.

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

   {{% alert title="Note" color="success" %}}

   In this example, two input sources and two subscribers are defined, and their triggering relationship is described as follows:

   - When input `eventB` is received, the input event is sent to the Knative service.
   - When input `eventB` and input `eventA` are received, the input event is sent to the metrics topic of the event bus and sent to the Knative service at the same time.

   {{% /alert %}}

2. Run the following commands to apply the configuration files.

   ```shell
   kubectl apply -f condition-trigger.yaml
   ```

3. Run the following commands to check the results.

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

4. Run the following command and you can see from the output that the `eventB` condition in the Trigger is matched and the Knative service is triggered because the event source `eventsource-b` is a cron task.

   ```shell
   $ kubectl get functions.core.openfunction.io
   NAME                                  BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
   sink                                  Skipped      Running                  serving-4x5wh   https://openfunction.io/default/sink   3h25m
   
   $ kubectl get po
   NAME                                                        READY   STATUS    RESTARTS   AGE
   serving-4x5wh-ksvc-wxbf2-v100-deployment-5c495c84f6-k2jdg   2/2     Running   0          46s
   ```

5. Create an event producer by referring to [Create an Event Producer](../event-bus-and-trigger#create-an-event-producer).

6. Run the following command and you can see from the output that the `eventA && eventB` condition in the Trigger is matched and the event is sent to the `metrics` topic of the event bus at the same time. The OpenFuncAsync function is triggered.

   ```shell
   $ kubectl get functions.core.openfunction.io
   NAME                                  BUILDSTATE   SERVINGSTATE   BUILDER   SERVING         URL                                   AGE
   trigger-target                        Skipped      Running                  serving-7hghp                                         103s
   
   $ kubectl get po
   NAME                                                        READY   STATUS    RESTARTS   AGE
   serving-7hghp-deployment-v100-z8wrf-946b4854d-svf55         2/2     Running   0          18s
   ```

