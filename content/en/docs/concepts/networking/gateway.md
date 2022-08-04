---
title: "Gateway"
linkTitle: "Gateway"
weight: 3420
description:
---

## Gateway Definition
An OpenFunction `Gateway` defines how users can access sync functions and how the Kubernetes `Gateway` resource should be used.
An OpenFunction `Gateway` is 1:1 with Kubernetes `Gateway`. 

Once a `Gateway` is created, the gateway controller will:
- If `gatewaySpec.listeners` does not contain the default listener named `ofn-http-internal`, it will be added to `gatewaySpec.listeners`.
- `gatewaySpec.listeners.[*].hostname` will be generated based on `domain` or `clusterDomain`.
- If `gatewayRef` is defined in a `Gateway`, the corresponding `gateway.networking.k8s.io` custom resource will be injected base on `gatewaySpec.listenters`.
- If `gatewayDef` is defined in a `Gateway`, a `gateway.networking.k8s.io` custom resource will be created based on `gatewaySpec.listenters`.
- Create a service named `gateway.openfunction.svc.cluster.local`, this service defines a unified entry for sync functions through `Gateway` within the cluster.

Once a `Gateway` is reconciled, the status of `Gateway` and `Gateway.gatewaySpec.listeners` will be shown in the `Gateway.status`.

## Default Gateway
By default, OpenFunction use `Contour` as Kubernetes `Gateway` implementation. Kubernetes `GatewayClass` and Kubernetes `Gateway` will be automatically created during installation. 
Then the following OpenFunction `Gateway` will be created:

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

You can edit the default `Gateway` to customize it:
```shell
kubectl edit gateway openfunction -n openfunction
```

## Custom Kubernetes Gateway implementation
The following documentation will show using istio as Kubernetes Gateway implementation.

1. Install OpenFunction without `Contour`:
```shell
helm install openfunction --set global.Contour.enabled=false openfunction/openfunction -n openfunction
```

2. Install `Istio` and enable its Knative integration:
```shell
kubectl apply -l knative.dev/crd-install=true -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/net-istio.yaml
```

3. Create `GatewayClass` named `istio`:
```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GatewayClass
metadata:
  name: istio
spec:
  controllerName: istio.io/gateway-controller
  description: The default Istio GatewayClass
```

4. Create OpenFunction `Gateway`:
```yaml
apiVersion: networking.openfunction.io/v1alpha1
kind: Gateway
metadata:
  name: custom-gateway
  namespace: openfunction
spec:
  domain: ofn.io
  clusterDomain: cluster.local
  hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"
  pathTemplate: "{{.Namespace}}/{{.Name}}"
  gatewayDef:
    namespace: openfunction
    gatewayClassName: istio
  gatewaySpec:
    listeners:
    - name: ofn-http-external
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
```

5. Reference custom `Gateway` via `gatewayRef` in `Function` definition:
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
      name: custom-gateway
      namespace: openfunction
```

## Multiple OpenFunction Gateway
Multiple `Gateway` are meaningless for OpenFunction, we currently only support one OpenFunction `Gateway`.
