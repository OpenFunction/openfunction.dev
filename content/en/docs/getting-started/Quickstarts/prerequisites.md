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

## Source repository Credential

If your source code is in a private git repository, you'll need to create a secret containing the private git repo's username and password:

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: git-repo-secret
  annotations:
    build.shipwright.io/referenced.secret: "true"
type: kubernetes.io/basic-auth
stringData:
  username: <cleartext username>
  password: <cleartext password>
EOF
```

You can then reference this secret in the `Function` CR's spec.build.srcRepo.credentials

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v2.0.0"
  image: "openfunctiondev/sample-go-func:v1"
  imageCredentials:
    name: push-secret
  port: 8080 # default to 8080
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "HelloWorld"
      FUNC_CLEAR_SOURCE: "true"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/knative/hello-world-go"
      revision: "main"
      credentials:
         name: git-repo-secret
  serving:
    template:
      containers:
        - name: function # DO NOT change this
          imagePullPolicy: IfNotPresent 
    runtime: "knative"
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
       version: 3.3.1
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
       cleanup.policy: delete
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

## WasmEdge

Function now supports using `WasmEdge` as workload runtime, here you can find steps to setup the `WasmEdge` workload runtime in a Kubernetes cluster (with `containerd` as the container runtime).

> You should run the following steps on all the nodes (or a subset of the nodes that will host the wasm workload) of your cluster.

1. **WasmEdge**

   The easiest way to install WasmEdge is to run the following command. Your system should have git and curl installed.
   ```shell
   wget -qO- https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local
   ```
   
2. **crun**
   
   The crun project has WasmEdge support baked in. For now, the easiest approach is just download the binary and move it to `/usr/local/bin/` 
   ```shell
   wget https://github.com/OpenFunction/OpenFunction/releases/latest/download/crun-linux-amd64
   mv crun-linux-amd64 /usr/local/bin/crun
   ```
   If the above approach does not work for you, please refer to [build and install a crun binary with WasmEdge support.](https://wasmedge.org/book/en/use_cases/kubernetes/container/crun.html)

3. **containerd**

   Edit the configuration `/etc/containerd/config.toml`, add the following section to setup crun runtime, make sure the BinaryName equal to your crun binary path
   ```
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.crun]
       runtime_type = "io.containerd.runc.v2"
       pod_annotations = ["*.wasm.*", "wasm.*", "module.wasm.image/*", "*.module.wasm.image", "module.wasm.image/variant.*"]
       privileged_without_host_devices = false
       [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.crun.options]
         BinaryName = "/usr/local/bin/crun"
   ```

   Restart containerd service:
   ```shell
   sudo systemctl restart containerd
   ```
