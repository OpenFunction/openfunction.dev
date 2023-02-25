---
title: "Wasm Functions"
linkTitle: "Wasm Functions"
weight: 3310
description:
---

`WasmEdge` is a lightweight, high-performance, and extensible WebAssembly runtime for cloud native, edge, and decentralized applications. It powers serverless apps, embedded functions, microservices, smart contracts, and IoT devices.

OpenFunction now supports building and running wasm functions with `WasmEdge` as the workload runtime.

> You can find the WasmEdge Integration proposal [here](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/20230223-wasmedge-integration.md)

## Wasm container images

The wasm image containing the wasm binary is a special container image without the OS layer. An special annotation `module.wasm.image/variant: compat-smart` should be added to this wasm container image for a wasm runtime like WasmEdge to recognize it. This is handled automatically in OpenFunction and users only need to specify the `workloadRuntime` as `wasmedge`. 

### The build phase of the wasm container images 

If `function.spec.workloadRuntime` is set to `wasmedge` or the function's annotation contains `module.wasm.image/variant: compat-smart`, 
`function.spec.build.shipwright.strategy` will be automatically generated based on the `ClusterBuildStrategy` named `wasmedge` in order to build a wasm container image with the `module.wasm.image/variant: compat-smart` annotation.

### The serving phase of the wasm container images

When `function.spec.workloadRuntime` is set to `wasmedge` or the function's annotation contains `module.wasm.image/variant: compat-smart`:
- If `function.spec.serving.annotations` does not contain `module.wasm.image/variant`, `module.wasm.image/variant: compat-smart` will be automatically added to `function.spec.serving.annotations`.
- If `function.spec.serving.template.runtimeClassName` is not set, this `runtimeClassName` will be automatically set to the default `openfunction-crun`

> If your kubernetes cluster is in a public cloud like `Azure`, you can set `spec.serving.template.runtimeClassName` manually to override the default `runtimeClassName`.

## Build and run wasm functions

> To setup `WasmEdge` workload runtime in kubernetes cluster and push images to a container registry,
> please refer to the [prerequisites](../../getting-started/Quickstarts/prerequisites) section for more info.

1. Create a wasm function

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: wasmedge-http-server
spec:
  workloadRuntime: wasmedge
  image: openfunctiondev/wasmedge_http_server:0.1.0
  imageCredentials:
    name: push-secret
  build:
    dockerfile: Dockerfile
    srcRepo:
      revision: main
      sourceSubPath: functions/knative/wasmedge/http-server
      url: https://github.com/OpenFunction/samples
  port: 8080
  route:
    rules:
      - matches:
          - path:
              type: PathPrefix
              value: /echo
  serving:
    runtime: knative
    scaleOptions:
      minReplicas: 0
    template:
      containers:
        - command:
            - /wasmedge_hyper_server.wasm
          imagePullPolicy: IfNotPresent
          livenessProbe:
            initialDelaySeconds: 3
            periodSeconds: 30
            tcpSocket:
              port: 8080
          name: function
EOF
```

2. Check the wasm function status

```shell
kubectl get functions.core.openfunction.io -w
NAME                   BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                      AGE
wasmedge-http-server   Succeeded    Running        builder-4p2qq   serving-lrd8c   http://wasmedge-http-server.default.svc.cluster.local/echo   12m
  ```

3. Access the wasm function

Once the `BUILDSTATE` becomes `Succeeded` and the `SERVINGSTATE` becomes `Running`, you can access this function through the address in the `ADDRESS` field:

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://wasmedge-http-server.default.svc.cluster.local/echo  -X POST -d "WasmEdge"
WasmEdge
```