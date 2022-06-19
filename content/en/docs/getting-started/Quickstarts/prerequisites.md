---
title: "Prerequisites"
linkTitle: "Prerequisites"
weight: 2210
description: 
---

## Registry Credential

When building a function, you'll need to push your function container image to a container registry like [Docker Hub](https://hub.docker.com/) or [Quay.io](https://quay.io/). To do that you'll need to generate a secret for your container registry first.

You can create this secret by filling in the `REGISTRY_SERVER`, `REGISTRY_USER` and `REGISTRY_PASSWORD` fields, and then run the following command.

```bash
REGISTRY_SERVER=https://index.docker.io/v1/
REGISTRY_USER=<your_registry_user>
REGISTRY_PASSWORD=<your_registry_password>
kubectl create secret docker-registry push-secret \
 --docker-server=$REGISTRY_SERVER \
 --docker-username=$REGISTRY_USER \
 --docker-password=$REGISTRY_PASSWORD
```

## Kafka

Async functions can be triggered by events in message queues like Kafka, here you can find steps to setup a Kafka cluster for demo purpose.

1. Install [strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator) in the default namespace.

   ```shell
   helm repo add strimzi https://strimzi.io/charts/
   helm install kafka-operator -n default strimzi/strimzi-kafka-operator
   ```

2. Run the following command to create a Kafka cluster and Kafka Topic in the default namespace. The Kafka and Zookeeper clusters created by this command have a storage type of **ephemeral** and are demonstrated using emptyDir.

   > Here we create a 1-replica Kafka server named `<kafka-server>` and a 1-replica topic named `<kafka-topic>` with 10 partitions

   ```shell
   cat <<EOF | kubectl apply -f -
   apiVersion: kafka.strimzi.io/v1beta2
   kind: Kafka
   metadata:
     name: <kafka-server>
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
     name: <kafka-topic>
     namespace: default
     labels:
       strimzi.io/cluster: <kafka-server>
   spec:
     partitions: 10
     replicas: 1
     config:
       retention.ms: 7200000
       segment.bytes: 1073741824
   EOF
   ```

3. Run the following command to check Pod status and wait for Kafka and Zookeeper to run and start.

   ```shell
   $ kubectl get po
   NAME                                              READY   STATUS        RESTARTS   AGE
   <kafka-server>-entity-operator-568957ff84-nmtlw   3/3     Running       0          8m42s
   <kafka-server>-kafka-0                            1/1     Running       0          9m13s
   <kafka-server>-zookeeper-0                        1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm         1/1     Running       0          11m
   ```

   Run the following command to view the metadata for the Kafka cluster.

   ```shell
   $ kafkacat -L -b <kafka-server>-kafka-brokers:9092
   ```