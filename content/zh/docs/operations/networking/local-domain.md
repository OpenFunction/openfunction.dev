---
title: "配置本地域名"
linkTitle: "配置本地域名"
weight: 4120
description:
---

## 配置本地域名
通过配置本地域名，您可以通过函数的外部地址从 Kubernetes 集群内部访问函数。

### 基于 `Gateway.spec.domain` 配置 `CoreDNS`
假设您有一个定义了此 `domain`：`*.ofn.io` 的 `Gateway`，您需要通过以下命令更新 `CoreDNS` 配置：
1. 编辑 `coredns` 配置映射：
```shell
kubectl -n kube-system edit cm coredns -o yaml
```
2. 在 `.:53` 部分的配置文件中添加 `rewrite stop name regex .*\.ofn\.io gateway.openfunction.svc.cluster.local`，例如：
```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        rewrite stop name regex .*\.ofn\.io gateway.openfunction.svc.cluster.local
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
...
```

### 基于 `Gateway.spec.domain` 配置 `nodelocaldns`
如果您也在使用 `nodelocaldns`，如 `Kubesphere`，您需要通过以下命令更新 `nodelocaldns` 配置：
1. 编辑 `nodelocaldns` 配置映射：
```shell
kubectl -n kube-system edit cm nodelocaldns -o yaml
```
2. 在配置文件中添加 `ofn.io:53` 部分，例如：
```yaml
apiVersion: v1
data:
  Corefile: |
    ofn.io:53 {
        errors
        cache {
            success 9984 30
            denial 9984 5
        }
        reload
        loop
        bind 169.254.25.10
        forward . 10.233.0.3 {
            force_tcp
        }
        prometheus :9253
    }
    cluster.local:53 {
        errors
        cache {
            success 9984 30
            denial 9984 5
        }
        reload
        loop
        bind 169.254.25.10
        forward . 10.233.0.3 {
            force_tcp
        }
        prometheus :9253
        health 169.254.25.10:9254
    }
    .:53 {
        errors
        cache 30
        reload
        loop
        bind 169.254.25.10
        forward . /etc/resolv.conf
        prometheus :9253
    }
...
```