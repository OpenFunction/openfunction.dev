---
title: "Function Definition"
linkTitle: "Function Definition"
weight: 3100
description: 
---
## Function

`Function` is the control plane of `Build` and `Serving` and it's also the interface for users to use OpenFunction. Users needn't to create the `Build` or `Serving` separately because `Function` is the only place to define a function's `Build` and `Serving`.

Once a function is created, it will controll the lifecycle of `Build` and `Serving` without user intervention:

- If `Build` is defined in a function, a builder custom resource will be created to build function's container image once a function is deployed.

- If `Serving` is defined in a function, a serving custom resource will be created to control a function's serving and autoscalling.

- `Build` and `Serving` can be defined together which means the function image will be built first and then it will be used in serving.

- `Build` can be defined without `Serving`, the function is used to build image only in this case.

- `Serving` can be defined without `Build`, the function will use a previously built function image for serving.

<div align=center><img width="80%" height="80%" src=/images/docs/en/introduction/what-is-openfunction/openfunction-0.5-architecture.svg/></div>

## Build

OpenFunction uses [Shipwright](https://github.com/shipwright-io/build) and [Cloud Native Buildpacks](https://buildpacks.io/) to build the function source code into container images.

Once a function is created with `Build` spec in it, a `builder` custom resource will be created which will use Shipwright to manage the build tools and strategy. The Shipwright will then use Tekton to control the process of building container images including fetching source code, generating image artifacts, and publishing images.

<div align=center><img width="60%" height="60%" src=/images/docs/en/concepts/function/openfunction-build.svg/></div>

## Serving

Once a function is created with `Serving` spec, a `Serving` custom resource will be created to control a function's serving phase. Currently OpenFunction Serving supports two runtimes: the Knative sync runtime and the OpenFunction async runtime.

## The sync runtime

For sync functions, OpenFunction currently supports using Knative Serving as runtime.
And we're planning to add another sync function runtime powered by the KEDA http-addon.

## The async runtime

OpenFunction's async runtime is an event-driven runtime which is implemented based on [KEDA](https://keda.sh/) and [Dapr](https://dapr.io/). Async functions can be triggered by various event types like message queue, cronjob, and MQTT etc.

<div align=center><img width="70%" height="70%" src=/images/docs/en/concepts/function/openfunction-serving.svg/></div>

## Reference

For more information, see [Function Specifications](../../reference/component-reference/function-spec).