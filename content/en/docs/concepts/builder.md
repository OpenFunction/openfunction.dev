---
title: "Builder"
linkTitle: "Builder"
weight: 15
description: >
    Concept of OpenFunction, a resource of the serverless framework, Builder
---

**Builder** defines the build work in OpenFunction for generating application images from source code.

Currently, OpenFunction Builder uses [Shipwright](#shipwright) and [Cloud Native Buildpacks](#cloud-native-buildpacks) to build application images. It controls the build process of application images through Shipwright, including acquiring code, generating image artefacts and publishing images through Cloud Native Buildpacks.

### Available builders

- [OpenFunction Builder](https://github.com/OpenFunction/builder)
- [GoogleCloudPlatform Builder](https://github.com/GoogleCloudPlatform/buildpacks)
- [Paketo Builder](https://paketo.io/docs/)

### Shipwright

[Shipwright](https://github.com/shipwright-io/build) is a scalable image pipeline framework for building container images on Kubernetes.

### Cloud Native Buildpacks

[Cloud Native Buildpacks](https://buildpacks.io/) is an OCI-standard image building framework that transforms source code into container image artefacts in custom steps (buildpack) without Dockerfiles.

### Reference

Builder is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [Builder CRD Spec]({{< ref "../reference/function-spec#buildimpl" >}}) for more information.

