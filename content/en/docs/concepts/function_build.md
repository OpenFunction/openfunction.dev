---
title: "Function Build"
linkTitle: "Function Build"
weight: 3110
description: 
---
Currently, OpenFunction supports building function images using [Cloud Native Buildpacks](https://buildpacks.io/) without the need to create a `Dockerfile`. 

In the meantime, you can also use OpenFunction to build [Serverless Applications](../serverless_apps/#build-and-run-a-serverless-application-with-a-dockerfile) with `Dockerfile`.

## Build functions by defining a build section

You can build your functions or applications from the source code in a git repo or from the source code stored locally.

### Build functions from source code in a git repo

You can build a function image by simply adding a build section in the `Function` definition like below.
If there is a serving section defined as well, the function will be launched as soon as the build completes.

```yaml
apiVersion: core.openfunction.io/v1beta2
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
      # FUNC_FRAMEWORK_VERSION: "v0.4.0"
      ## Use FUNC_GOPROXY to set the goproxy
      # FUNC_GOPROXY: "https://goproxy.cn"
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "functions/async/logs-handler-function/"
      revision: "main"
```

> To push the function image to a container registry, you have to create a secret containing the registry's credential and add the secret to `imageCredentials`.
> You can refer to the [prerequisites](../../getting-started/Quickstarts/prerequisites) for more info.

### Build functions from local source code

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
apiVersion: core.openfunction.io/v1beta2
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
cd samples/functions/knative/hello-world-go
```

### Build the function image with the pack CLI

```shell
pack build func-helloworld-go --builder openfunction/builder-go:v2.4.0-1.17 --env FUNC_NAME="HelloWorld"  --env FUNC_CLEAR_SOURCE=true
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

|           | Builders                                                                                                                           |
|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| Go        | openfunction/builder-go:v2.4.0 (openfunction/builder-go:latest)                                                                    |
| Nodejs    | openfunction/builder-node:v2-16.15 (openfunction/builder-node:latest)                                                              |
| Java      | openfunction/builder-java:v2-11, openfunction/builder-java:v2-16, openfunction/builder-java:v2-17, openfunction/builder-java:v2-18 |
| Python    | openfunction/gcp-builder:v1                                                                                                        |
| DotNet    | openfunction/gcp-builder:v1                                                                                                        |