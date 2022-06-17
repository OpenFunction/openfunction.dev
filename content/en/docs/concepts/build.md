---
title: "Build"
linkTitle: "Build"
weight: 3200
description: >	
    Get familiar with how a function is built in OpenFunction
---

## Build

OpenFunction uses [Shipwright](https://github.com/shipwright-io/build) and [Cloud Native Buildpacks](https://buildpacks.io/) to build the function source code into container images.

Once a function is created with `Build` spec in it, a `builder` custom resource will be created which will use Shipwright to manage the build tools and strategy. The Shipwright will then use Tekton to control the process of building container images including fetching source code, generating image artifacts, and publishing images.

<div align=center><img width="60%" height="60%" src=/images/docs/en/concepts/function/openfunction-build.svg/></div>

