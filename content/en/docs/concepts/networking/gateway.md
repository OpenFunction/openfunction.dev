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

After a `Gateway` is deployed, you'll be able to find the status of `Gateway` and `Gateway.gatewaySpec.listeners` in the `Gateway.status` field, e.g:
```yaml
status:
  conditions:
  - lastTransitionTime: "2022-08-04T10:20:57Z"
    message: Gateway is scheduled
    observedGeneration: 2
    reason: Scheduled
    status: "True"
    type: Scheduled
  - lastTransitionTime: "2022-08-04T10:20:57Z"
    message: Valid Gateway
    observedGeneration: 2
    reason: Valid
    status: "True"
    type: Ready
  listeners:
  - attachedRoutes: 0
    conditions:
    - lastTransitionTime: "2022-08-04T10:20:57Z"
      message: Valid listener
      observedGeneration: 2
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-internal
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
  - attachedRoutes: 0
    conditions:
    - lastTransitionTime: "2022-08-04T10:20:57Z"
      message: Valid listener
      observedGeneration: 2
      reason: Ready
      status: "True"
      type: Ready
    name: ofn-http-external
    supportedKinds:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
```

## Default Gateway
By default, OpenFunction use `Contour` as Kubernetes `Gateway` implementation. Kubernetes `GatewayClass` and Kubernetes `Gateway` will be automatically created during installation.
The following OpenFunction `Gateway` will also be created automatically:
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
The following documentation will demonstrate using Istio as the underlying Kubernetes Gateway implementation.

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
```shell
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GatewayClass
metadata:
  name: istio
spec:
  controllerName: istio.io/gateway-controller
  description: The default Istio GatewayClass
EOF
```

4. Create OpenFunction `Gateway`:
```shell
kubectl apply -f - <<EOF
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
EOF
```

5. Reference custom `Gateway` via `gatewayRef` in `Function` definition:
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
      name: custom-gateway
      namespace: openfunction
EOF
```

## Multiple OpenFunction Gateway
Multiple `Gateway` are meaningless for OpenFunction, we currently only support one OpenFunction `Gateway`.
