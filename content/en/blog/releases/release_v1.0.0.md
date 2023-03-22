---
title: "Announcing OpenFunction 1.0.0: Integrate WasmEdge to support Wasm Functions and Enhanced CI/CD"
linkTitle: "Release v1.0.0"
date: 2023-03-11
weight: 92

---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) is a cloud-native open-source FaaS (Function as a Service) platform aiming to let you focus on your business logic only. Today, we are thrilled to announce the general availability of OpenFunction 1.1.0.

In this update, we have continued our commitment to providing developers with more flexible and powerful tools, and have added some new feature. This release integrates WasmEdge to support Wasm functions; we have also enhanced the CI/CD functionality of OpenFunction to provide relatively complete end-to-end CI/CD functionality; and we have added the ability to build an image of a function or application directly from local code in this release, making it easier for developers to publish and deploy their code.

We have optimized OpenFunction's performance and stability and fixed bugs to improve the user experience.

The following introduces the major updates.

## Integrate WasmEdge to support Wasm Functions

WasmEdge is a lightweight, high-performance, and scalable WebAssembly runtime for cloud-native, edge, and decentralized applications. It powers serverless apps, embedded functions, microservices, smart contracts, and IoT devices.

OpenFunction now supports building and running wasm functions with WasmEdge as the workload runtime. WasmEdge has been an alternative container runtime of Docker, Containerd, and CRI-O.

### Create a wasm function

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: wasmedge-http-server
spec:
  workloadRuntime: wasmedge
  image: openfunctiondev/wasmedge_http_server:0.1.0
  imageCredentials:
    name: push-secret
  build:
    dockerfile: Dockerfile
    srcRepo:
      revision: main
      sourceSubPath: functions/knative/wasmedge/http-server
      url: https://github.com/OpenFunction/samples
  port: 8080
  route:
    rules:
      - matches:
          - path:
              type: PathPrefix
              value: /echo
  serving:
    runtime: knative
    scaleOptions:
      minReplicas: 0
    template:
      containers:
        - command:
            - /wasmedge_hyper_server.wasm
          imagePullPolicy: IfNotPresent
          livenessProbe:
            initialDelaySeconds: 3
            periodSeconds: 30
            tcpSocket:
              port: 8080
          name: function
EOF
```

With the WasmEdge engine, developers can write and run functions using a variety of Wasm-enabled languages and development frameworks.

> Please refer to the official documentation [Wasm Functions](https://openfunction.dev/docs/concepts/wasm_functions/).

## Enhanced CI/CD

Previously users can use OpenFunction to build function or application source code into container images, and then the system deploys the built image directly to the underlying sync/async Serverless runtime without user intervention.

But OpenFunction can neither rebuild the image and then redeploy it whenever the function or application source code changes nor redeploy the image whenever this image changes (When the image is built and pushed manually or in another function)

Starting from v1.0.0, OpenFunction adds the ability to detect source code or image changes and then rebuild and/or redeploy the newly built image in a new component called Revision Controller. The Revision Controller is able to:

- Detect source code changes in GitHub, GitLab or Gitee, and then rebuild and redeploy the new built image whenever the source code changes.
- Detect the bundle container image (image containing the source code) changes, then rebuild and redeploy the new built image whenever the bundle image changes.
- Detect the function or application image changes, then redeploy the new image whenever the function or application image changes.

The enhanced CI/CD functionality ensures that the code runs efficiently in different environments, and users can have better control over the versions and code quality during the development and deployment process. This also provides a better user experience.

> Please refer to the official documentation [CI/CD](https://openfunction.dev/docs/concepts/cicd/).

## Build functions from local source code

To build functions or applications from local source code, you'll need to package your local source code into a container image and push this image to a container registry. 

Suppose your source code is in the `samples` directory, you can use the following `Dockerfile` to build a source code bundle image.

```shell
FROM scratch
WORKDIR /
COPY samples samples/
```

Then you can build the source code bundle image like this:

```shell
docker build -t <your registry name>/sample-source-code:latest -f </path/to/the/dockerfile> .
docker push <your registry name>/sample-source-code:latest
```

> It's recommended to use the empty image `scratch` as the base image to build the source code bundle image, a non-empty base image may cause the source code copy to fail.

Unlike defining the `spec.build.srcRepo.url` field for the git repo method, you'll need to define the `spec.build.srcRepo.bundleContainer.image` field instead.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: logs-async-handler
spec:
  build:
    srcRepo:
      bundleContainer:
        image: openfunctiondev/sample-source-code:latest
      sourceSubPath: "/samples/functions/async/logs-handler-function/"
```

> The `sourceSubPath` is the absolute path of the source code in the source code bundle image.

## Other enhancements

In addition to the major changes mentioned above, this release has the following changes and enhancements.

- OpenFunction
  - The core v1alpha2 API was deprecated and removed
  - Add sha256 to serving image
  - Add information of build source to function status
  - Bump shipwright to v0.11.0, knative to v0.32.0, dapr to v1.8.3, and go to 1.18
- functions-framework-java released version 1.0.0
  - Support multiple functions in one pod
  - Support for automatic publishing
- Builder
  - Support multiple functions in one pod
  - Update the default java framework version to 1.0.0
- revision-controller released version 1.0.0


These are the main feature changes in OpenFunction v1.0.0 and we would like to thank all contributors for their contributions. If you are looking for an efficient and flexible cloud-native function development platform, OpenFunction v1.0.0 is the perfect choice for you.

For more details and documentation, please visit our website and GitHub repo.

- [Website](https://openfunction.dev/)：https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.0.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.0.0
