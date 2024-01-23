---
title: "函数入口"
linkTitle: "函数入口"
weight: 3440
description:
---

有几种方法可以访问同步函数。我们将在以下部分详细说明。

> 本文档假设你正在使用默认的 [OpenFunction 网关](../gateway/#the-default-openfunction-gateway) 并且你有一个名为 `function-sample` 的同步函数。

## 从集群内部访问函数
### 通过内部地址访问函数
OpenFunction 会为每个同步 `Function` 创建此服务：`{{.Name}}.{{.Namespace}}.svc.cluster.local`。此服务将用于提供函数的内部地址。

通过运行以下命令获取 `Function` 内部地址：
```shell
export FUNC_INTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="Internal")].value}')
```

这个地址提供了在集群内部访问函数的默认方法，适合用作 `EventSource` 的 `sink.url`。

在 pod 中使用 curl 访问 `Function`：
```shell=
kubectl run --rm ofn-test -i --tty --image=radial/busyboxplus:curl -- curl -sv $FUNC_INTERNAL_ADDRESS
```

## 从集群外部访问函数
### 通过 Kubernetes 网关的 IP 地址访问函数
获取 Kubernetes 网关的 ip 地址：
```shell
export IP=$(kubectl get node -l "! node.kubernetes.io/exclude-from-external-load-balancers" -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
```

获取函数的 HOST 和 PATH：
```shell
export FUNC_HOST=$(kubectl get function function-sample -o=jsonpath='{.status.route.hosts[0]}')
export FUNC_PATH=$(kubectl get function function-sample -o=jsonpath='{.status.route.paths[0].value}')
```

直接使用 curl 访问 `Function`：
```shell
curl -sv -HHOST:$FUNC_HOST http://$IP$FUNC_PATH
```

### 通过外部地址访问函数

要通过外部地址访问同步函数，你需要先配置 DNS。Magic DNS 或真实 DNS 都可以：

- Magic DNS

  获取 Kubernetes 网关的 ip 地址：
  ```shell
  export IP=$(kubectl get node -l "! node.kubernetes.io/exclude-from-external-load-balancers" -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
  ```
  用 Magic DNS 替换 OpenFunction 网关中定义的域：
  ```shell
  export DOMAIN="$IP.sslip.io"
  kubectl patch gateway.networking.openfunction.io/openfunction -n openfunction --type merge --patch '{"spec": {"domain": "'$DOMAIN'"}}'
  ```
  然后，你可以在 `Function` 的状态字段中看到 `Function` 外部地址：
  ```shell
  kubectl get function function-sample -oyaml
  ```
  ```yaml
  status:
    addresses:
    - type: External
      value: http://function-sample.default.172.31.73.53.sslip.io/
    - type: Internal
      value: http://function-sample.default.svc.cluster.local/
    build:
      resourceHash: "14903236521345556383"
      state: Skipped
    route:
      conditions:
      - message: Valid HTTPRoute
        reason: Valid
        status: "True"
        type: Accepted
      hosts:
      - function-sample.default.172.31.73.53.sslip.io
      - function-sample.default.svc.cluster.local
      paths:
      - type: PathPrefix
        value: /
    serving:
      lastSuccessfulResourceRef: serving-t56fq
      resourceHash: "2638289828407595605"
      resourceRef: serving-t56fq
      service: serving-t56fq-ksvc-bv8ng
      state: Running
  ```

- Real DNS

  如果你有一个外部 IP 地址，你可以配置一个通配符 A 记录作为你的域：
  ```
  # 这里 example.com 是 OpenFunction 网关中定义的域
  *.example.com == A <external-ip>
  ```
  如果你有一个 CNAME，你可以配置一个 CNAME 记录作为你的域：
  ```
  # 这里 example.com 是 OpenFunction 网关中定义的域
  *.example.com == CNAME <external-name>
  ```
  用你上面配置的域替换 OpenFunction 网关中定义的域：
  ```shell
  export DOMAIN="example.com"
  kubectl patch gateway.networking.openfunction.io/openfunction -n openfunction --type merge --patch '{"spec": {"domain": "'$DOMAIN'"}}'
  ```
  然后，你可以在 `Function` 的状态字段中看到 `Function` 外部地址：
  ```shell
  kubectl get function function-sample -oyaml
  ```
  ```yaml
  status:
    addresses:
    - type: External
      value: http://function-sample.default.example.com/
    - type: Internal
      value: http://function-sample.default.svc.cluster.local/
    build:
      resourceHash: "14903236521345556383"
      state: Skipped
    route:
      conditions:
      - message: Valid HTTPRoute
        reason: Valid
        status: "True"
        type: Accepted
      hosts:
      - function-sample.default.example.com
      - function-sample.default.svc.cluster.local
      paths:
      - type: PathPrefix
        value: /
    serving:
      lastSuccessfulResourceRef: serving-t56fq
      resourceHash: "2638289828407595605"
      resourceRef: serving-t56fq
      service: serving-t56fq-ksvc-bv8ng
      state: Running
  ```

然后，你可以通过运行以下命令获取 `Function` 外部地址：
```shell
export FUNC_EXTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="External")].value}')
```
现在，你可以直接使用 curl 访问 `Function`：

```shell=
curl -sv $FUNC_EXTERNAL_ADDRESS
```