---
title: "Function Entrypoints"
linkTitle: "Function Entrypoints"
weight: 3440
description:
---

There are several methods to access a sync function. Let's elaborate on this in the following section.

> This documentation will assume you are using default [OpenFunction Gateway](../gateway/#the-default-openfunction-gateway) and you have a sync function named `function-sample`.

## Access functions from within the cluster
### Access functions by the internal address
OpenFunction will create this service for every sync `Function`: `{{.Name}}.{{.Namespace}}.svc.cluster.local`. This service will be used to provide the Function internal address.

Get `Function` internal address by running following command:
```shell
export FUNC_INTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="Internal")].value}')
```

This address provides the default method for accessing functions within the cluster, it's suitable for use as `sink.url` of `EventSource`.

Access `Function` using curl in pod:
```shell=
kubectl run --rm ofn-test -i --tty --image=radial/busyboxplus:curl -- curl -sv $FUNC_INTERNAL_ADDRESS
```

## Access functions from outside the cluster
### Access functions by the Kubernetes Gateway's IP address
Get Kubernetes Gateway's ip address:
```shell
export IP=$(kubectl get node -l "! node.kubernetes.io/exclude-from-external-load-balancers" -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
```
Get Function's HOST and PATH:
```shell
export FUNC_HOST=$(kubectl get function function-sample -o=jsonpath='{.status.route.hosts[0]}')
export FUNC_PATH=$(kubectl get function function-sample -o=jsonpath='{.status.route.paths[0].value}')
```

Access `Function` using curl directly:
```shell
curl -sv -HHOST:$FUNC_HOST http://$IP$FUNC_PATH
```

### Access functions by the external address

To access a sync function by the external address, you'll need to configure DNS first. Either Magic DNS or real DNS works:

- Magic DNS

  Get Kubernetes Gateway's ip address:
  ```shell
  export IP=$(kubectl get node -l "! node.kubernetes.io/exclude-from-external-load-balancers" -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
  ```
  Replace domain defined in OpenFunction Gateway with Magic DNS:
  ```shell
  export DOMAIN="$IP.sslip.io"
  kubectl patch gateway.networking.openfunction.io/openfunction -n openfunction --type merge --patch '{"spec": {"domain": "'$DOMAIN'"}}'
  ```
  Then, you can see `Function` external address in `Function`'s status field:
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
    - lastTransitionTime: "2022-08-20T11:03:14Z"
      message: Valid HTTPRoute
      observedGeneration: 13
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

  If you have an external IP address, you can configure a wildcard A record as your domain:
  ```
  # Here example.com is the domain defined in OpenFunction Gateway
  *.example.com == A <external-ip>
  ```
  If you have a CNAME, you can configure a CNAME record as your domain:
  ```
  # Here example.com is the domain defined in OpenFunction Gateway
  *.example.com == CNAME <external-name>
  ```
  Replace domain defined in OpenFunction Gateway with the domain you configured above:
  ```shell
  export DOMAIN="example.com"
  kubectl patch gateway.networking.openfunction.io/openfunction -n openfunction --type merge --patch '{"spec": {"domain": "'$DOMAIN'"}}'
  ```
  Then, you can see `Function` external address in `Function`'s status field:
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
    - lastTransitionTime: "2022-08-20T13:07:17Z"
      message: Valid HTTPRoute
      observedGeneration: 14
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

Then, you can get `Function` external address by running following command:
```shell
export FUNC_EXTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="External")].value}')
```

Now, you can access `Function` using curl directly:

```shell=
curl -sv $FUNC_EXTERNAL_ADDRESS
```