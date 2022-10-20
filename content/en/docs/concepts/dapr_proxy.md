---
title: "Dapr Proxy"
linkTitle: "Dapr Proxy"
weight: 3400
description:
---

## Overview

Previously, OpenFunction used dapr service in `sidecar` mode, which caused problems such as function startup time and
overhead affected by dapr sidecar.

So we decided to implement `dapr-proxy` to solve these problems, in `dapr-proxy` mode all pods of a function share one dapr service.

> You can find the dapr-proxy proposal [here](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/20220919-dapr-proxy.md).

The following diagram illustrates how function pods share the dapr service and how proxy forwarding events to function pods:

![](https://i.imgur.com/4r6jm8x.png)

## Dapr service mode

We use two annotations defined in `spec.serving.annotations` to control how the dapr service is used: `openfunction.io/enable-dapr`
and `openfunction.io/dapr-service-mode`.

The control logic:

- when `"openfunction.io/enable-dapr"` is `true` and `openfunction.io/dapr-service-mode` is `standalone`, enable
  `dapr-proxy`.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample-serving-only
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    annotations:
      openfunction.io/enable-dapr: "true"
      openfunction.io/dapr-service-mode: "standalone"
    runtime: knative
```

- when `openfunction.io/enable-dapr` is `true` and `openfunction.io/dapr-service-mode` is `sidecar`, enable
  `dapr-sidecar`.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample-serving-only
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    annotations:
      openfunction.io/enable-dapr: "true"
      openfunction.io/dapr-service-mode: "sidecar"
    runtime: knative
```

- when `openfunction.io/enable-dapr` is `false`, disable both `dapr-proxy` and `dapr-sidecar`.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample-serving-only
spec:
  version: "v1.0.0"
  image: "openfunctiondev/v1beta1-http:latest"
  serving:
    annotations:
      openfunction.io/enable-dapr: "false"
    runtime: knative
```

> The default value of these annotations:
> - when user does not define `openfunction.io/enable-dapr` in `spec.serving.annotations`, if user
  defined `spec.serving.inputs` or `spec.serving.outputs`,
  the value of this annotation will be treated as `true`, otherwise it will be treated as `false`.
> - when user does not define `openfunction.io/dapr-service-mode`, the default value of this annotation is `standalone`.