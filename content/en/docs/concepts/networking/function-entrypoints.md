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

### Access functions by the external address
> The Function external address is usually used for external access, but you can also use it within the cluster for development or testing.
You should configure local domain first, see [Configure Local Domain](../../../operations/networking/local-domain).

Get `Function` external address by running following command:
```shell
export FUNC_EXTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="External")].value}')
```

Access `Function` using curl in pod:
```shell=
kubectl run --rm ofn-test -i --tty --image=radial/busyboxplus:curl -- curl -sv $FUNC_EXTERNAL_ADDRESS
```
## Access functions from outside the cluster
### Access functions by the Kubernetes Gateway's IP address
Get Kubernetes Gateway's ip address:
```shell
export IP=$(kubectl get node -l node-role.kubernetes.io/worker= -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
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

#### Configure DNS
You have two options to configure DNS:

- Option 1: Real DNS

    The real DNS configuration is based on the network topology of your environment.

- Option 2: Configure the local hosts file

    Get Function's HOSTNAME and Gateway's ip:
    ```shell
    export HOSTNAME="function-sample.default.ofn.io"
    export IP=$(kubectl get node -l node-role.kubernetes.io/worker= -o=jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
    ```
    
    Append this line to /etc/hosts:
    ```shell
    echo $IP  $HOSTNAME | sudo tee -a /etc/hosts
    ```

#### Access Function
Get `Function` external address by running following command:
```shell
export FUNC_EXTERNAL_ADDRESS=$(kubectl get function function-sample -o=jsonpath='{.status.addresses[?(@.type=="External")].value}')
```
Access `Function` using curl directly:

```shell=
curl -sv $FUNC_EXTERNAL_ADDRESS
```