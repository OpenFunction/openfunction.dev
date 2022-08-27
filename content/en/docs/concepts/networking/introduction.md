---
title: "Introduction"
linkTitle: "Introduction"
weight: 3410
description: 
---

## Overview

Previously starting from v0.5.0, OpenFunction uses Kubernetes Ingress to provide unified entrypoints for sync functions, and a nginx ingress controller has to be installed.

With the maturity of [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/), we decided to implement OpenFunction Gateway based on the [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) to replace the previous ingress based domain method in OpenFunction v0.7.0.

>You can find the OpenFunction Gateway proposal [here](https://github.com/OpenFunction/OpenFunction/blob/main/docs/proposals/202207-openfunction-gateway.md)

OpenFunction Gateway provides a more powerful and more flexible function gateway including features like:

- Enable users to switch to any [gateway implementations](https://gateway-api.sigs.k8s.io/implementations/) that support [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) such as Contour, Istio, Apache APISIX, Envoy Gateway (in the future) and more in an easier and vendor-neutral way.

- Users can choose to install a default gateway implementation (Contour) and then define a new `gateway.networking.k8s.io` or use any existing gateway implementations in their environment and then reference an existing `gateway.networking.k8s.io`.

- Allow users to customize their own function access pattern like `hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"` for host-based access.

- Allow users to customize their own function access pattern like `pathTemplate: "{{.Namespace}}/{{.Name}}"` for path-based access.

- Allow users to customize each function's route rules (host-based, path-based or both) in function definition and default route rules are provided for each function if there're no customized route rules defined.

- Send traffic to Knative service revisions directly without going through Knative’s own gateway anymore. You will need only OpenFunction Gateway since OpenFunction 0.7.0 to access OpenFunction sync functions, and you can ignore Knative’s domain config errors if you do not need to access Knative service directly.

- Traffic splitting between function revisions (in the future)

The following diagram illustrates how client traffics go through OpenFunction Gateway and then reach a function directly:

<div align=center><img width="100%" height="100%" src=/images/docs/en/concepts/gateway/gateway.png/></div>