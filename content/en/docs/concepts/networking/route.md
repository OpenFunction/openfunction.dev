---
title: "Route"
linkTitle: "Route"
weight: 3430
description:
---

## Route
`route` is part of `Function` resource. `route` defines how traffic from the `Gateway` listener is routed to function.
`route` specify the `Gateway` they want to attach to using `GatewayRef`, this will allow the `route` to receive traffic from the `Gateway`.

Once a synchronous `Function` is created, the function controller will:
- If `route.gatewayRef` is not defined in function, will look for `Gateway` called `openfunction` in `openfunction` namespace, then attach to this `Gateway`.
- If `route.hostnames` is not defined in function, will be automatically generated based on `Gateway.spec.hostTemplate`.
- If `route.rules` is not defined in function, will be automatically generated based on `Gateway.spec.pathTemplate` or path of `/`.
- a `HTTPRoute` custom resource will be created based on `route`. `BackendRefs` will be automatically link to the corresponding Knative service revision 
and label `HTTPRouteLabelKey` will be added to this `HTTPRoute`.
- Create service `{{.Name}}.{{.Namespace}}.svc.cluster.local`, this service defines an entry for the function to access within the cluster.
- If the `Gateway` referenced by `route.gatewayRef` changed, will update the `HTTPRoute`.

Once a synchronous `Function` is reconciled, the `Function` addresses and status of `route` will be shown in the `Function.status`.

## Host-Based Routing
`host-based` is the default routing mode. When `route.hostnames` are not defined in function,
function controller will generate `route.hostnames` based on `gateway.spec.hostTemplate`. 
If `route.rules` is also not defined, will generate `route.rules` based on path of `/`.

```yaml
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
```

> If you are using default OpenFunction [Gateway](https://openfunction.dev/docs/concepts/networking/gateway/#default-gateway), the function external address will be as follows:
```
http://function-sample.default.ofn.io/
```

## Path-Based Routing
You can define specific `route.hostnames`, then function controller will only generate `route.rules` based on `gateway.spec.pathTemplate`.

```yaml
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
```

> If you are using the default OpenFunction [Gateway](https://openfunction.dev/docs/concepts/networking/gateway/#default-gateway), the function's external address will be as below:
```
http://sample.default.ofn.io/default/function-sample/
```

## Define both `route.hostnames` and `route.rules`
You can define both `route.hostnames` and `route.rules` to customize how traffic should be routed to a function.

{{% alert title="Note" color="success" %}}

In this mode, you need to resolve possible conflicts between HTTPRoutes yourself.

{{% /alert %}}
```yaml
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
```

> If you are using default OpenFunction [Gateway](https://openfunction.dev/docs/concepts/networking/gateway/#default-gateway), the function external address will be as follows:
```
http://sample.default.ofn.io/v2/foo/
```
