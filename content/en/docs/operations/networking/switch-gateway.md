---
title: "Switch to another Kubernetes Gateway"
linkTitle: "Switch to another Kubernetes Gateway"
weight: 4110
description:
---

You can switch to any [gateway implementations](https://gateway-api.sigs.k8s.io/implementations/) that support [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) such as Contour, Istio, Apache APISIX, Envoy Gateway (in the future) and more in an easier and vendor-neutral way.

For example, you can choose to use Istio as the underlying `Kubernetes Gateway` like this:

1. Install OpenFunction without `Contour`:

```shell
helm install openfunction --set global.Contour.enabled=false openfunction/openfunction -n openfunction
```

2. Install `Istio` and then enable its Knative integration:

```shell
kubectl apply -l knative.dev/crd-install=true -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/net-istio.yaml
```

3. Create a `GatewayClass` named `istio`:
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

4. Create an `OpenFunction Gateway`:

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

5. Reference the custom `OpenFunction Gateway` (Istio) in the `gatewayRef` field of a `Function`:

```shell
kubectl apply -f - <<EOF
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    template:
      containers:
        - name: function
          imagePullPolicy: Always
    triggers:
      http:
        route:
          gatewayRef:
            name: custom-gateway
            namespace: openfunction
EOF
```