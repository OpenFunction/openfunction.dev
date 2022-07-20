---
title: "Function Build"
linkTitle: "Function Build"
weight: 3110
description: 
---
Currently, OpenFunction builds function images with [Cloud Native Buildpacks](https://buildpacks.io/). The traditional `Dockerfile` build method will be supported in the future.

## Build functions by adding a build section in the function definition

You can build a function image by simply adding a build section in the `Function` definition like below.
If there is a serving section defined as well, the function will be launched as soon as the build completes.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: logs-async-handler
spec:
  version: "v2.0.0"
  image: openfunctiondev/logs-async-handler:v1
  imageCredentials:
    name: push-secret
  build:
    builder: openfunction/builder-go:latest
    env:
      FUNC_NAME: "LogsHandler"
      FUNC_CLEAR_SOURCE: "true"
      ## Customize functions framework version, valid for functions-framework-go for now
      ## Usually you needn't to do so because the builder will ship with the latest functions-framework
      # FUNC_FRAMEWORK_VERSION: "v0.3.0"
      # Use FUNC_GOPROXY to set the goproxy
      # FUNC_GOPROXY: "https://goproxy.cn"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/async/logs-handler-function/"
      revision: "main"
```

> To push the function image to a container registry, you have to create a secret containing the registry's credential and add the secret to `imageCredentials`.
> You can refer to the [prerequisites](../../getting-started/Quickstarts/prerequisites) for more info.

## Build functions with the pack CLI

Usually it's necessary to build function images directly from local source code especially for debug purpose or for offline environment. You can use the pack CLI for this.

Pack is a tool maintained by the Cloud Native Buildpacks project to support the use of buildpacks.
It enables the following functionality:

- Build an application using buildpacks.
- Rebase application images created using buildpacks.
- Creation of various components used within the ecosystem.

Follow the instructions [here](https://buildpacks.io/docs/tools/pack/) to install the `pack` CLI tool.
You can find more details on how to use the pack CLI [here](https://buildpacks.io/docs/tools/pack/cli/pack/).

To build OpenFunction function images from source code locally, you can follow the steps below:

### Download function samples

```shell
git clone https://github.com/OpenFunction/samples.git
cd samples/functions/Knative/hello-world-go
```

### Build the function image with the pack CLI

```shell
pack build func-helloworld-go --builder openfunction/builder-go:v2.3.0-1.16 --env FUNC_NAME="HelloWorld"  --env FUNC_CLEAR_SOURCE=true
```

### Launch the function image locally

```shell
docker run --rm --env="FUNC_CONTEXT={\"name\":\"HelloWorld\",\"version\":\"v1.0.0\",\"port\":\"8080\",\"runtime\":\"Knative\"}" --env="CONTEXT_MODE=self-host" --name func-helloworld-go -p 8080:8080 func-helloworld-go
```

### Visit the function

```shell
curl http://localhost:8080
```

Output example:

```shell
hello, world!
```

## OpenFunction Builders

To build a function image with [Cloud Native Buildpacks](https://buildpacks.io/), a builder image is needed.

Here you can find builders for popular languages maintained by the OpenFunction community:

|           | Builders |
|-----------|----------|
| Go        | openfunction/builder-go:v2.3.0 (openfunction/builder-go:latest) |
| Nodejs    | openfunction/builder-node:v2-16.15 (openfunction/builder-node:latest) |
| Java      | openfunction/builder-java:v2-11, openfunction/builder-java:v2-16, openfunction/builder-java:v2-17, openfunction/builder-java:v2-18 |
| Python    | openfunction/gcp-builder:v1 |
| DotNet    | openfunction/gcp-builder:v1 |