---
title: "Build Strategy"
linkTitle: "Build Strategy"
weight: 3120
description:
---

Build Strategy is used to control the build process. There are two types of strategies, `ClusterBuildStrategy` and `BuildStrategy`. 
Both strategies define a group of steps necessary to control the application build process.

`ClusterBuildStrategy` is cluster-wide, while `BuildStrategy` is namespaced.

There are 4 built-in `ClusterBuildStrategy` in OpenFunction, you can find more details in the following sections.

## openfunction

The `openfunction` ClusterBuildStrategy uses [Buildpacks](https://buildpacks.io/docs/) to build function images which is the default build strategy.

The following are the parameters of the `openfunction` ClusterBuildStrategy:

| Name | Type | Describe |  
| --- | --- | --- |
| RUN_IMAGE | string | Reference to a run image to use |  
| CACHE_IMAGE | string | Cache Image is a way to preserve cache layers across different builds, which can improve build performance when building functions or applications with lots of dependencies like Java functions. |  
| BASH_IMAGE | string | The bash image that the strategy used.  |  
| ENV_VARS  | string | Environment variables to set during _build-time_. The formate is `key1=value1,key2=value2`. |

Users can set these parameters like this:

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: logs-async-handler
spec:
  build:
    shipwright:
      params:
        RUN_IMAGE: ""
        ENV_VARS: ""
```

## buildah

The `buildah` ClusterBuildStrategy uses [buildah](https://buildah.io/) to build application images. 

To use `buildah` ClusterBuildStrategy, you can define a `Function` like this:

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: sample-go-app
  namespace: default
spec:
  build:
    builder: openfunction/buildah:v1.23.1
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: buildah
```

The following are the parameters of the `buildah` ClusterBuildStrategy:

| Name | Type | Describe |  Default |
| --- | --- | --- | --- |
| registry-search   | string | The registries for searching short name images such as `golang:latest`, separated by commas. | docker.io,quay.io |  
| registry-insecure | string | The fully-qualified name of insecure registries. An insecure registry is a registry that does not have a valid SSL certificate or only supports HTTP. |
| registry-block    | string | The registries that need to block pull access. | "" |

## kaniko

The `kaniko` ClusterBuildStrategy uses [kaniko](https://github.com/GoogleContainerTools/kaniko) to build application images.

To use `kaniko` ClusterBuildStrategy, you can define a `Function` like this:

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-kaniko
  namespace: default
spec:
  build:
    builder: openfunction/kaniko-executor:v1.7.0
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: kaniko
```

## ko

The `ko` ClusterBuildStrategy uses [ko](https://github.com/ko-build/ko) to build `Go` application images.

To use `ko` ClusterBuildStrategy, you can define a `Function` like this:

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-ko
  namespace: default
spec:
  build:
    builder: golang:1.17
    dockerfile: Dockerfile
    shipwright:
      strategy:
        kind: ClusterBuildStrategy
        name: ko
```

The following are the parameters of `ko` ClusterBuildStrategy:

| Name | Type | Describe |  Default |
| --- | --- | --- | --- |
| go-flags          | string | Value for the GOFLAGS environment variable. | "" |  
| ko-version        | string | Version of ko, must be either 'latest', or a release name from https://github.com/google/ko/releases. | "" |
| package-directory | string | The directory inside the context directory containing the main package. | "." |

## Custom Strategy

Users can customize their own strategy. To customize strategy, you can refer to [this](https://github.com/shipwright-io/build/blob/main/docs/buildstrategies.md).