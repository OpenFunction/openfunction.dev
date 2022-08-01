---
title: "Configure Local Domain"
linkTitle: "Configure Local Domain"
weight: 3450
description:
---

## Configure Local Domain
By configuring the local domain, you can access functions within kubernetes cluster through the Function external address.
### Configure `CoreDNS` based on `gateway.spec.domain`
Assume you have a `gateway` that defines this `domain`: `*.ofn.io`, you need to update `CoreDNS` configuration via following commands:
1. Edit the `coredns` configmap:
```shell=
kubectl -n kube-system edit cm coredns -o yaml
```
2. Add `rewrite stop name regex .*\.ofn\.io gateway.openfunction.svc.cluster.local` to the configuration file in the `.:53` section, e.g:
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
### Configure `nodelocaldns` based on `gateway.spec.domain`
If you are also using `nodelocaldns`, such as the `kubesphere` user, you need update `nodelocaldns` configuration via following commands:
1. Edit the `nodelocaldns` configmap:
```shell=
kubectl -n kube-system edit cm nodelocaldns -o yaml
```
2. Add `ofn.io:53` section to the configuration file, e.g:
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