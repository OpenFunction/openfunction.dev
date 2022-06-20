---
title: "Build OpenFunction Locally"
linkTitle: "Build OpenFunction Locally"
weight: 7210
description:
---

This document describes how to build OpenFunction locally.

## Prerequisites

- You need to have a Kubernetes cluster and a Go development environment. If not ready, [install your Kubernetes cluster](https://kubesphere.io/blogs/install-kubernetes-using-kubekey/) and [set the Go environment up](https://go.dev/doc/code) first.

  {{% alert title="Note" color="success" %}}

  - Make sure the Kubernetes version is 1.18+ and Go version is 1.12+.
  - Make sure that your GOPATH and PATH have been configured in accordance with the Go environment instructions.

  {{% /alert %}}

- OpenFunction uses [Go Modules](https://github.com/golang/go/wiki/Modules) to manage dependencies.

- OpenFunction components are often deployed as containers in Kubernetes. If you need to rebuild the OpenFunction components in your Kubernetes cluster, you have to [install Docker](https://docs.docker.com/install/) in advance.

{{% alert title="Note" color="success" %}}

If you are interested in developing controller-manager, see [sample-controller](https://github.com/kubernetes/sample-controller) and [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder).

{{% /alert %}}

## Build Image Locally

1. Fork [the OpenFunction repository](https://github.com/OpenFunction/OpenFunction) and clone it to your local development environment.
2. Modify the `Dockerfile` in the root directory based on your needs.
3. Use Docker to build your `openfunction` image and push it to your personal image repository.

## Use Your Image

1. After pushing your `openfunction` image, run the following command to modify the YAML file of `openfunction-controller-manager`.

   ```shell
   kubectl edit deployments.apps -n openfunction openfunction-controller-manager
   ```

2. Change the value of `spec.template.spec.containers.image` to the image URL of your `openfunction` image.

2. Save the changes and wait for `openfunction-controller-manager` to be up and running again.

   
