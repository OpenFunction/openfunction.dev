---
title: "Announcing General Availability of OpenFunction 0.7.0"
linkTitle: "Release v0.7.0"
date: 2022-08-19
weight: 94

---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) is a cloud-native open-source FaaS (Function as a Service) platform aiming to let you focus on your business logic only. Today, we are thrilled to announce the general availability of OpenFunction 0.7.0.

Thanks to your contributions and feedback, OpenFunction 0.7.0 brings more exciting features like OpenFunction Gateway, async and sync functions of Java and Node.js, and Helm installation. Meantime, OpenFunction dependencies have been upgraded for better user experience.

### OpenFunction Gateway

Starting from OpenFunction 0.5.0, OpenFunction uses Kubernetes Ingress to provide a unified entry for sync functions, and an Nginx ingress controller must be installed.

With the maturity of the [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/), we decided to implement OpenFunction Gateway based on the Kubernetes Gateway API to replace the previous ingress-based domain method in OpenFunction 0.7.0.

OpenFunction Gateway provides a more powerful and more flexible function gateway, including the following features:

- Enable users to switch to any [gateway implementations](https://gateway-api.sigs.k8s.io/implementations/) that support [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) such as Contour, Istio, Apache APISIX, Envoy Gateway (in the future) and more in an easier and vendor-neutral way.

- Users can choose to install a default gateway implementation (Contour) and then define a new `gateway.networking.k8s.io` or use any existing gateway implementations in their environment and then reference an existing `gateway.networking.k8s.io`.

- Allow users to customize their own function access pattern like `hostTemplate: "{{.Name}}.{{.Namespace}}.{{.Domain}}"` for host-based access.

- Allow users to customize their own function access pattern like `pathTemplate: "{{.Namespace}}/{{.Name}}"` for path-based access.

- Allow users to customize each function's route rules (host-based, path-based or both) in the function definition and default route rules are provided for each function if there're no customized route rules defined.

- Access functions from outside of the cluster. You only need to configure a valid domain name in OpenFunction Gateway. Both Magic DNS and Real DNS are supported.

- Send traffic to Knative service revisions directly without going through Knative’s own gateway anymore. You will need only OpenFunction Gateway since OpenFunction 0.7.0 to access OpenFunction sync functions, and you can ignore Knative’s domain config errors if you do not need to access Knative service directly.

In upcoming releases, OpenFunction will support traffic splitting between function revisions.

### Support for multiple programming languages

The OpenFunction community has been striving to support more programming languages:

- Go

  [functions-framework-go](https://github.com/OpenFunction/functions-framework-go) 0.4.0 supports the ability to define multiple sub-functions in one function and call the sub-functions through different Paths and Methods.

- Java
  
  [functions-framework-java](https://github.com/OpenFunction/functions-framework-java) supports async and sync functions.

- Node.js

  [functions-framework-nodejs](https://github.com/OpenFunction/functions-framework-nodejs) 0.5.0 supports async and sync functions, and the async functions can be triggered by the sync functions.

  OpenFunction will introduce more features in [functions-framework-nodejs](https://github.com/OpenFunction/functions-framework-nodejs) 0.6.0, such as add-ons and integration with SkyWalking.

OpenFunction will support more programming languages, such as Python and Dotnet.

### Use Helm to install OpenFunction and its dependencies

_Previously, we use CLI to install OpenFunction and its dependencies. Now, this method has been depreciated._

In OpenFunction 0.7.0., we use Helm to install OpenFunction and its dependencies, which is more cloud-native and solves the problem that some users are unable to access Google Container Registry (gcr.io).

**TL;DR**
```
helm repo add openfunction https://openfunction.github.io/charts/
helm repo update
helm install openfunction openfunction/openfunction -n openfunction --create-namespace
```

### Upgrade dependencies

| Components             | OpenFunction 0.6.0 | OpenFunction 0.7.0 |
| ---------------------- |--------------------|--------------------|
| Knative Serving        | 1.0.1              | 1.3.2              |
| Dapr                   | 1.5.1              | 1.8.3              |
| Keda                   | 2.4.0              | 2.7.1              |
| Shipwright             | 0.6.1              | 0.10.0             |
| Tekton Pipelines       | 0.30.0             | 0.37.0             |
