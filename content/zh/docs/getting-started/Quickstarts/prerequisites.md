---
title: "先决条件"
linkTitle: "先决条件"
weight: 2210
description:
---

## 镜像仓库凭证

在构建函数时，您需要将函数容器镜像推送到像[Docker Hub](https://hub.docker.com/)或[Quay.io](https://quay.io/)这样的容器镜像仓库。为此，您首先需要为您的容器镜像仓库生成一个密钥。

您可以通过填写`REGISTRY_SERVER`，`REGISTRY_USER`和`REGISTRY_PASSWORD`字段，然后运行以下命令来创建此密钥。

```bash
REGISTRY_SERVER=https://index.docker.io/v1/
REGISTRY_USER=<your_registry_user>
REGISTRY_PASSWORD=<your_registry_password>
kubectl create secret docker-registry push-secret \
 --docker-server=$REGISTRY_SERVER \
 --docker-username=$REGISTRY_USER \
 --docker-password=$REGISTRY_PASSWORD
```

## 源代码库凭证

如果您的源代码位于私有git仓库中，您需要创建一个包含私有git仓库的用户名和密码的密钥：

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

然后，您可以在`Function` CR的spec.build.srcRepo.credentials中引用此密钥

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

异步函数可以由消息队列中的事件触发，如Kafka，这里您可以找到设置Kafka集群的步骤，用于演示目的。

1. 在默认命名空间中安装[strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator)。

   ```shell
   helm repo add strimzi https://strimzi.io/charts/
   helm install kafka-operator -n default strimzi/strimzi-kafka-operator
   ```

2. 运行以下命令在默认命名空间中创建Kafka集群和Kafka Topic。此命令创建的Kafka和Zookeeper集群的存储类型为**ephemeral**，并使用emptyDir进行演示。

   > 这里我们创建一个名为`<kafka-server>`的1副本Kafka服务器和一个名为`<kafka-topic>`的1副本主题，带有10个分区

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

3. 运行以下命令检查Pod状态，并等待Kafka和Zookeeper运行并启动。

   ```shell
   $ kubectl get po
   NAME                                              READY   STATUS        RESTARTS   AGE
   <kafka-server>-entity-operator-568957ff84-nmtlw   3/3     Running       0          8m42s
   <kafka-server>-kafka-0                            1/1     Running       0          9m13s
   <kafka-server>-zookeeper-0                        1/1     Running       0          9m46s
   strimzi-cluster-operator-687fdd6f77-cwmgm         1/1     Running       0          11m
   ```

   运行以下命令查看Kafka集群的元数据。

   ```shell
   $ kafkacat -L -b <kafka-server>-kafka-brokers:9092
   ```

## WasmEdge

函数现在支持使用`WasmEdge`作为工作负载运行时，这里您可以找到在Kubernetes集群中设置`WasmEdge`工作负载运行时的步骤（以`containerd`为容器运行时）。

> 您应在集群的所有节点（或将承载wasm工作负载的节点的子集）上运行以下步骤。

### 步骤1：安装WasmEdge

安装WasmEdge的最简单方法是运行以下命令。您的系统应已安装git和curl。
   ```shell
   wget -qO- https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local
   ```

### 步骤2：安装容器运行时

#### crun

crun项目已经内置了WasmEdge支持。现在，最简单的方法就是下载二进制文件并将其移动到`/usr/local/bin/`
```shell
wget https://github.com/OpenFunction/OpenFunction/releases/latest/download/crun-linux-amd64
mv crun-linux-amd64 /usr/local/bin/crun
```
如果上述方法对您不起作用，请参考[构建并安装带有WasmEdge支持的crun二进制文件。](https://wasmedge.org/book/en/use_cases/kubernetes/container/crun.html)

### 步骤3：设置CRI运行时

#### 选项1：containerd

> 您可以按照此[安装指南](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#installing-containerd)来安装containerd，并按照此[设置指南](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#setting-up-containerd-for-kubernetes)来为Kubernetes设置containerd。

首先，编辑配置`/etc/containerd/config.toml`，添加以下部分来设置crun运行时，确保BinaryName等于您的crun二进制路径
```
# Add crun runtime here
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.crun]
  runtime_type = "io.containerd.runc.v2"
  pod_annotations = ["*.wasm.*", "wasm.*", "module.wasm.image/*", "*.module.wasm.image", "module.wasm.image/variant.*"]
  privileged_without_host_devices = false
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.crun.options]
    BinaryName = "/usr/local/bin/crun"
```

接下来，重启containerd服务：

```shell
sudo systemctl restart containerd
```

#### 选项2：CRI-O

> 您可以按照此[安装指南](https://github.com/cri-o/cri-o/blob/main/install.md)来安装CRI-O，并按照此[设置指南](https://github.com/cri-o/cri-o/blob/main/tutorials/kubernetes.md#running-cri-o-on-kubernetes-cluster)来为Kubernetes设置CRI-O。

CRI-O默认使用runc运行时，我们需要配置它来使用crun。这是通过添加到两个配置文件来完成的。

首先，创建一个`/etc/crio/crio.conf`文件，并添加以下行作为其内容。它告诉CRI-O默认使用crun。

```
[crio.runtime]
default_runtime = "crun"
```

crun运行时反过来在`/etc/crio/crio.conf.d/01-crio-runc.conf`文件中定义。
```
[crio.runtime.runtimes.runc]
runtime_path = "/usr/lib/cri-o-runc/sbin/runc"
runtime_type = "oci"
runtime_root = "/run/runc"
# The above is the original content

# Add crun runtime here
[crio.runtime.runtimes.crun]
runtime_path = "/usr/local/bin/crun"
runtime_type = "oci"
runtime_root = "/run/crun"
```

接下来，重启CRI-O以应用配置更改。

```shell
systemctl restart crio
```
