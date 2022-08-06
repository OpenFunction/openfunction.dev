---
title: "Route"
linkTitle: "Route"
weight: 3430
description:
---

## What is `Route`?

`Route` is part of the `Function` definition. `Route` defines how traffic from the `Gateway` listener is routed to a function. 

A `Route` specifies the `Gateway` to which it will attach in `GatewayRef` which allow it to receive traffic from the `Gateway`.

Once a sync `Function` is created, the function controller will:

- Look for the`Gateway` called `openfunction` in `openfunction` namespace, then attach to this `Gateway` if `route.gatewayRef` is not defined in the function.
- Automatically generate `route.hostnames` based on `Gateway.spec.hostTemplate`, if `route.hostnames` is not defined in function.
- Automatically generate `route.rules` based on `Gateway.spec.pathTemplate` or path of `/`, if `route.rules` is not defined in function.
- a `HTTPRoute` custom resource will be created based on `Route`. `BackendRefs` will be automatically link to the corresponding Knative service revision 
and label `HTTPRouteLabelKey` will be added to this `HTTPRoute`.
- Create service `{{.Name}}.{{.Namespace}}.svc.cluster.local`, this service defines an entry for the function to access within the cluster.
- If the `Gateway` referenced by `route.gatewayRef` changed, will update the `HTTPRoute`.

After a sync `Function` is deployed, you'll be able to find `Function` addresses and `Route` status in `Function`'s status field, e.g:

```yaml
status:
  addresses:
    - type: External
      value: http://function-sample-serving-only.default.ofn.io/
    - type: Internal
      value: http://function-sample-serving-only.default.svc.cluster.local/
  build:
    resourceHash: "14903236521345556383"
    state: Skipped
  route:
    conditions:
      - lastTransitionTime: "2022-08-04T10:43:29Z"
        message: Valid HTTPRoute
        observedGeneration: 1
        reason: Valid
        status: "True"
        type: Accepted
    hosts:
      - function-sample-serving-only.default.ofn.io
      - function-sample-serving-only.default.svc.cluster.local
    paths:
      - type: PathPrefix
        value: /
  serving:
    lastSuccessfulResourceRef: serving-znk54
    resourceHash: "10715302888241374768"
    resourceRef: serving-znk54
    service: serving-znk54-ksvc-nbg6f
    state: Running
```

{{% alert title="Note" color="success" %}}

The Address of type `External` in `Funtion.status` provides the default method for accessing functions from within the cluster.
This internal address is not affected by the `Gateway` referenced by `route.gatewayRef` and it's suitable for use as `sink.url` of `EventSource`.

The Address of type `Internal` in `Funtion.status` provides the method for accessing functions from outside the cluster.
This external address is generated based on `route.gatewayRef`, `router.hostnames` and `route.rules`. The routing mode only takes effect on this external address, The following documentation will explain how it works.

For more information about how to access functions, see [Access functions](../../../concepts/networking/access-functions/).

{{% /alert %}}

## Host Based Routing

`Host-based` is the default routing mode. When `route.hostnames` is not defined,
`route.hostnames` will be generated based on `gateway.spec.hostTemplate`. 
If `route.rules` is not defined, `route.rules` will be generated based on path of `/`.

```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  port: 8080
  serving:
    runtime: knative
    template:
      containers:
        - name: function
          imagePullPolicy: Always
  route:
    gatewayRef:
      name: openfunction
      namespace: openfunction
EOF
```

> If you are using the default [OpenFunction Gateway](../gateway#the-default-openfunction-gateway), the function external address will be as below:

```shell
http://function-sample.default.ofn.io/
```

## Path Based Routing

If you define `route.hostnames` in a function, `route.rules` will be generated based on `gateway.spec.pathTemplate`.

```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  port: 8080
  serving:
    runtime: knative
    template:
      containers:
        - name: function
          imagePullPolicy: Always
  route:
    gatewayRef:
      name: openfunction
      namespace: openfunction
    hostnames:
    - "sample.ofn.io"
EOF
```

> If you are using the default [OpenFunction Gateway](../gateway#the-default-openfunction-gateway), the function external address will be as below:

```shell
http://sample.default.ofn.io/default/function-sample/
```

## Host and Path based routing

You can define hostname and path at the same time to customize how traffic should be routed to your function.

{{% alert title="Note" color="success" %}}

In this mode, you'll need to resolve possible conflicts between HTTPRoutes by yourself.

{{% /alert %}}
```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  port: 8080
  serving:
    runtime: knative
    template:
      containers:
        - name: function
          imagePullPolicy: Always
  route:
    gatewayRef:
      name: openfunction
      namespace: openfunction
    rules:
      - matches:
          - path:
              type: PathPrefix
              value: /v2/foo
    hostnames:
    - "sample.ofn.io"
EOF
```

> If you are using the default [OpenFunction Gateway](../gateway#the-default-openfunction-gateway), the function external address will be as below:

```shell
http://sample.default.ofn.io/v2/foo/
```
