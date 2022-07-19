---
title: "Elastic Log Alerting"
linkTitle: "Elastic Log Alerting"
weight: 5400
description: >	
    Learn how to create an async function to find out error logs.
---

This document describes how to create an async function to find out error logs.

## Overview

This document uses an asynchronous function to analyze the log stream in Kafka to find out the error logs. The async function will then send alerts to Slack. The following diagram illustrates the entire workflow.

![](/images/docs/en/best-practices/logs-handler-function/elastic-log-processing.png)

## Prerequisites

- You have [installed OpenFunction](../../getting-started/installation/).
- You have [created a secret](../../getting-started/your-first-function/function-go/#create-a-secret).

## Create a Kafka Server and Topic

1. Run the following commands to install [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator) in the default namespace.

   ```shell
   helm repo add strimzi https://strimzi.io/charts/
   helm install kafka-operator -n default strimzi/strimzi-kafka-operator
   ```

2. Use the following content to create a file `kafka.yaml`.

   ```yaml
   apiVersion: kafka.strimzi.io/v1beta2
   kind: Kafka
   metadata:
     name: kafka-logs-receiver
     namespace: default
   spec:
     kafka:
       version: 3.1.0
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

3. Run the following command to deploy a 1-replica Kafka server named `kafka-logs-receiver` and 1-replica Kafka topic named `logs` in the default namespace.

   ```shell
   kubectl apply -f kafka.yaml
   ```

4. Run the following command to check pod status and wait for Kafka and Zookeeper to be up and running.

   ```shell
   $ kubectl get po
   NAME                                                     READY   STATUS        RESTARTS   AGE
   kafka-logs-receiver-entity-operator-57dc457ccc-tlqqs     3/3     Running       0          8m42s
   kafka-logs-receiver-kafka-0                              1/1     Running       0          9m13s
   kafka-logs-receiver-zookeeper-0                          1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm                1/1     Running       0          11m
   ```

5. Run the following commands to view the metadata of the Kafka cluster.

   ```shell
   # Starts a utility pod.
   $ kubectl run utils --image=arunvelsriram/utils -i --tty --rm
   # Checks metadata of the Kafka cluster.
   $ kafkacat -L -b kafka-logs-receiver-kafka-brokers:9092
   ```

## Create a Logs Handler Function

1. Use the following example YAML file to create a manifest `logs-handler-function.yaml` and modify the value of `spec.image` to set your own image registry address.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: logs-async-handler
   spec:
     version: "v2.0.0"
     image: <your registry name>/logs-async-handler:latest
     imageCredentials:
       name: push-secret
     build:
       builder: openfunction/builder-go:latest
       env:
         FUNC_NAME: "LogsHandler"
         FUNC_CLEAR_SOURCE: "true"
         # Use FUNC_GOPROXY to set the goproxy
         # FUNC_GOPROXY: "https://goproxy.cn"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/async/logs-handler-function/"
         revision: "main"
     serving:
       runtime: "async"
       scaleOptions:
         keda:
           scaledObject:
             pollingInterval: 15
             minReplicaCount: 0
             maxReplicaCount: 10
             cooldownPeriod: 60
             advanced:
               horizontalPodAutoscalerConfig:
                 behavior:
                   scaleDown:
                     stabilizationWindowSeconds: 45
                     policies:
                     - type: Percent
                       value: 50
                       periodSeconds: 15
                   scaleUp:
                     stabilizationWindowSeconds: 0
       triggers:
         - type: kafka
           metadata:
             topic: logs
             bootstrapServers: kafka-server-kafka-brokers.default.svc.cluster.local:9092
             consumerGroup: logs-handler
             lagThreshold: "20"
       template:
         containers:
           - name: function
             imagePullPolicy: Always
       inputs:
         - name: kafka
           component: kafka-receiver
       outputs:
         - name: notify
           component: notification-manager
           operation: "post"
       bindings:
         kafka-receiver:
           type: bindings.kafka
           version: v1
           metadata:
             - name: brokers
               value: "kafka-server-kafka-brokers:9092"
             - name: authRequired
               value: "false"
             - name: publishTopic
               value: "logs"
             - name: topics
               value: "logs"
             - name: consumerGroup
               value: "logs-handler"
         notification-manager:
           type: bindings.http
           version: v1
           metadata:
             - name: url
               value: http://notification-manager-svc.kubesphere-monitoring-system.svc.cluster.local:19093/api/v2/alerts
   ```

2. Run the following command to create the function `logs-async-handler`.

   ```shell
   kubectl apply -f logs-handler-function.yaml
   ```

3. The logs handler function will be triggered by messages from the logs topic in Kafka.
