---
title: "切换到其他 Kubernetes Gateway 实现"
linkTitle: "切换到其他 Kubernetes Gateway 实现"
weight: 4110
description:
---

您可以以更简单、厂商中立的方式切换到任何支持 [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) 的 [网关实现](https://gateway-api.sigs.k8s.io/implementations/)，如 Contour、Istio、Apache APISIX、Envoy Gateway（未来）等。

例如，您可以选择使用 Istio 作为底层的 `Kubernetes Gateway`，如下所示：

1. 安装 OpenFunction，但不包括 `Contour`：

```shell
helm install openfunction --set global.Contour.enabled=false openfunction/openfunction -n openfunction
```

2. 安装 `Istio`，然后启用其 Knative 集成：

```shell
kubectl apply -l knative.dev/crd-install=true -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.3.0/net-istio.yaml
```

3. 创建一个名为 `istio` 的 `GatewayClass`：
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

4. 创建一个 `OpenFunction Gateway`：

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

5. 在 `Function` 的 `gatewayRef` 字段中引用自定义的 `OpenFunction Gateway`（Istio）：

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