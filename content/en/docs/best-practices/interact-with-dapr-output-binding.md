---
title: "Create a Knative-based Function to Interact with Middleware"
linkTitle: "Create a Knative-based Function to Interact with Middleware"
weight: 5200
description: >	
    Learn how to create a Knative-based function to interact with middleware via Dapr components.
---

This document describes how to create a Knative-based function to interact with middleware via Dapr components.

## Overview

Similar to asynchronous functions, the functions that are based on Knative runtime can interact with middleware through Dapr components. This document uses two functions, `function-front` and `kafka-input`, for demonstration.

The following diagram illustrates the relationship between these functions.

![](/images/docs/en/best-practices/interact-with-dapr-output-binding/knative-dapr.png)

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
     name: kafka-server
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
     name: sample-topic
     namespace: default
     labels:
       strimzi.io/cluster: kafka-server
   spec:
     partitions: 10
     replicas: 1
     config:
       retention.ms: 7200000
       segment.bytes: 1073741824
   ```

3. Run the following command to deploy a 1-replica Kafka server named `kafka-server` and 1-replica Kafka topic named `sample-topic` in the default namespace.

   ```shell
   kubectl apply -f kafka.yaml
   ```

4. Run the following command to check pod status and wait for Kafka and Zookeeper to be up and running.

   ```shell
   $ kubectl get po
   NAME                                              READY   STATUS        RESTARTS   AGE
   kafka-server-entity-operator-568957ff84-nmtlw     3/3     Running       0          8m42s
   kafka-server-kafka-0                              1/1     Running       0          9m13s
   kafka-server-zookeeper-0                          1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm         1/1     Running       0          11m
   ```

5. Run the following commands to view the metadata of the Kafka cluster.

   ```shell
   # Starts a utility pod.
   $ kubectl run utils --image=arunvelsriram/utils -i --tty --rm
   # Checks metadata of the Kafka cluster.
   $ kafkacat -L -b kafka-server-kafka-brokers:9092
   ```

## Create Functions

1. Use the following example YAML file to create a manifest `kafka-input.yaml` and modify the value of `spec.image` to set your own image registry address. The field `spec.serving.inputs` defines an input source that points to a Dapr component of the Kafka server. It means that the `kafka-input` function will be driven by events in the topic `sample-topic` of the Kafka server.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: kafka-input
   spec:
     version: "v1.0.0"
     image: <your registry name>/kafka-input:latest
     imageCredentials:
       name: push-secret
     build:
       builder: openfunction/builder-go:latest
       env:
         FUNC_NAME: "HandleKafkaInput"
         FUNC_CLEAR_SOURCE: "true"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/async/bindings/kafka-input"
         revision: "main"
     serving:
       runtime: async
       scaleOptions:
         minReplicas: 0
         maxReplicas: 10 
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
             topic: sample-topic
             bootstrapServers: kafka-server-kafka-brokers.default.svc:9092
             consumerGroup: kafka-input
             lagThreshold: "20"
       inputs:
         - name: greeting
           component: target-topic
       bindings:
         target-topic:
           type: bindings.kafka
           version: v1
           metadata:
             - name: brokers
               value: "kafka-server-kafka-brokers:9092"
             - name: topics
               value: "sample-topic"
             - name: consumerGroup
               value: "kafka-input"
             - name: publishTopic
               value: "sample-topic"
             - name: authRequired
               value: "false"
       template:
         containers:
           - name: function
             imagePullPolicy: Always
   ```
   
2. Run the following command to create the function `kafka-input`.

   ```shell
   kubectl apply -f kafka-input.yaml
   ```

3. Use the following example YAML file to create a manifest `function-front.yaml` and modify the value of `spec.image` to set your own image registry address.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: function-front
     annotations:
       plugins: |
         pre:
         - plugin-custom
         - plugin-example
         post:
         - plugin-custom
         - plugin-example
   spec:
     version: "v1.0.0"
     image: "<your registry name>/sample-knative-dapr:latest"
     imageCredentials:
       name: push-secret
     port: 8080 # Default to 8080
     build:
       builder: openfunction/builder-go:latest
       env:
         FUNC_NAME: "ForwardToKafka"
         FUNC_CLEAR_SOURCE: "true"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/knative/with-output-binding"
         revision: "main"
     serving:
       scaleOptions:
         minReplicas: 0
         maxReplicas: 5
       runtime: knative
       outputs:
         - name: target
           component: kafka-server
           operation: "create"
       bindings:
         kafka-server:
           type: bindings.kafka
           version: v1
           metadata:
             - name: brokers
               value: "kafka-server-kafka-brokers:9092"
             - name: authRequired
               value: "false"
             - name: publishTopic
               value: "sample-topic"
             - name: topics
               value: "sample-topic"
             - name: consumerGroup
               value: "function-front"
       template:
         containers:
           - name: function
             imagePullPolicy: Always
   ```
   
   {{% alert title="Note" color="success" %}}

   `metadata.plugins.pre` defines the order of plugins that need to be called before the user function is executed. `metadata.plugins.post` defines the order of plugins that need to be called after the user function is executed. For more information about the logic of these two plugins and the effect of the plugins after they are executed, see [Plugin mechanism](https://github.com/OpenFunction/samples/blob/main/functions-framework/README.md#plugin-mechanism).

   {{% /alert %}}

4. In the manifest, `spec.serving.outputs` defines an output that points to a Dapr component of the Kafka server. That allows you to send custom content to the output `target` in the function `function-front`.

   ```go
   func Sender(ctx ofctx.Context, in []byte) (ofctx.Out, error) {
     ...
   	_, err := ctx.Send("target", greeting)
   	...
   }
   ```

5. Run the following command to create the function `function-front`.

   ```shell
   kubectl apply -f function-front.yaml
   ```

## Check Results

1. Run the following command to view the status of the functions.

   ```shell
   $ kubectl get functions.core.openfunction.io
   
   NAME             BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         URL                                             AGE
   function-front   Succeeded    Running        builder-bhbtk   serving-vc6jw   http://openfunction.io/default/function-front   2m41s
   kafka-input      Succeeded    Running        builder-dprfd   serving-75vrt                                                   2m21s
   ```

   {{% alert title="Note" color="success" %}}

   The `URL`, provided by the OpenFunction Domain, is the address that can be accessed. To access the function through this URL address, you need to make sure that DNS can resolve this address.

   {{% /alert %}}

2. Run the following command to create a pod in the cluster for accessing the function.

   ```shell
   kubectl run curl --image=radial/busyboxplus:curl -i --tty --rm
   ```

3. Run the following command to access the function through `URL`.

   ```shell
   [ root@curl:/ ]$ curl -d '{"message":"Awesome OpenFunction!"}' -H "Content-Type: application/json" -X POST http://openfunction.io.svc.cluster.local/default/function-front
   ```

4. Run the following command to view the log of `function-front`.

   ```shell
   kubectl logs -f \
     $(kubectl get po -l \
     openfunction.io/serving=$(kubectl get functions function-front -o jsonpath='{.status.serving.resourceRef}') \
     -o jsonpath='{.items[0].metadata.name}') \
     function
   ```

   The output looks as follows.

   ```shell
   dapr client initializing for: 127.0.0.1:50001
   I0125 06:51:55.584973       1 framework.go:107] Plugins for pre-hook stage:
   I0125 06:51:55.585044       1 framework.go:110] - plugin-custom
   I0125 06:51:55.585052       1 framework.go:110] - plugin-example
   I0125 06:51:55.585057       1 framework.go:115] Plugins for post-hook stage:
   I0125 06:51:55.585062       1 framework.go:118] - plugin-custom
   I0125 06:51:55.585067       1 framework.go:118] - plugin-example
   I0125 06:51:55.585179       1 knative.go:46] Knative Function serving http: listening on port 8080
   2022/01/25 06:52:02 http - Data: {"message":"Awesome OpenFunction!"}
   I0125 06:52:02.246450       1 plugin-example.go:83] the sum is: 2
   ```

5. Run the following command to view the log of `kafka-input`.

   ```shell
   kubectl logs -f \
     $(kubectl get po -l \
     openfunction.io/serving=$(kubectl get functions kafka-input -o jsonpath='{.status.serving.resourceRef}') \
     -o jsonpath='{.items[0].metadata.name}') \
     function
   ```

   The output looks as follows.

   ```shell
   dapr client initializing for: 127.0.0.1:50001
   I0125 06:35:28.332381       1 framework.go:107] Plugins for pre-hook stage:
   I0125 06:35:28.332863       1 framework.go:115] Plugins for post-hook stage:
   I0125 06:35:28.333749       1 async.go:39] Async Function serving grpc: listening on port 8080
   message from Kafka '{Awesome OpenFunction!}'
   ```

