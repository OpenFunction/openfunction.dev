---
title: "OpenFunction Gateway"
linkTitle: "OpenFunction Gateway"
weight: 3420
description:
---

## Inside OpenFunction Gateway

Backed by the [Kubernetes Gateway](https://gateway-api.sigs.k8s.io/), an `OpenFunction Gateway` defines how users can access sync functions.

Whenever an `OpenFunction Gateway` is created, the gateway controller will:

- Add a default listener named `ofn-http-internal` to `gatewaySpec.listeners` if there isn't one there.

- Generate `gatewaySpec.listeners.[*].hostname` based on `domain` or `clusterDomain`.

- Inject `gatewaySpec.listenters` to the existing `Kubernetes Gateway` defined by the `gatewayRef` of the `OpenFunction Gateway`.

- Create an new `Kubernetes Gateway` based on the `gatewaySpec.listenters` field in `gatewayDef` of the `OpenFunction Gateway`.

- Create a service named `gateway.openfunction.svc.cluster.local` that defines a unified entry for sync functions.

After an `OpenFunction Gateway` is deployed, you'll be able to find the status of `Kubernetes Gateway` and its `listeners` in `OpenFunction Gateway` status:

```yaml
status:
  conditions:
  - message: Gateway is scheduled
    reason: Scheduled
    status: "True"
    type: Scheduled
  - message: Valid Gateway
    reason: Valid
    status: "True"
    type: Ready
  listeners:
  - attachedRoutes: 0
    conditions:
    - message: Valid listener
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-internal
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
  - attachedRoutes: 0
    conditions:
    - message: Valid listener
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-external
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
```

## The Default OpenFunction Gateway

`OpenFunction Gateway` uses `Contour` as the default `Kubernetes Gateway` implementation.
The following `OpenFunction Gateway` will be created automatically once you install OpenFunction:

```yaml
apiVersion: networking.openfunction.io/v1alpha1
kind: Gateway
metadata:
  name: openfunction
  namespace: openfunction
spec:
  domain: ofn.io
  clusterDomain: cluster.local
  hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"
  pathTemplate: "{{.Namespace}}/{{.Name}}"
  httpRouteLabelKey: "app.kubernetes.io/managed-by"
  gatewayRef:
    name: contour
    namespace: projectcontour
  gatewaySpec:
    listeners:
      - name: ofn-http-internal
        hostname: "*.cluster.local"
        protocol: HTTP
        port: 80
        allowedRoutes:
          namespaces:
            from: All
      - name: ofn-http-external
        hostname: "*.ofn.io"
        protocol: HTTP
        port: 80
        allowedRoutes:
          namespaces:
            from: All
```

You can customize the default `OpenFunction Gateway` like below:
```shell
kubectl edit gateway openfunction -n openfunction
```

## Switch to a different Kubernetes Gateway

You can switch to any [gateway implementations](https://gateway-api.sigs.k8s.io/implementations/) that support [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) such as Contour, Istio, Apache APISIX, Envoy Gateway (in the future) and more in an easier and vendor-neutral way. 

> [Here](../../../operations/networking/switch-gateway) you can find more details.

## Multiple OpenFunction Gateway
Multiple `Gateway` are meaningless for OpenFunction, we currently only support one OpenFunction `Gateway`.
