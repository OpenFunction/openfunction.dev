---
title: "CI/CD"
linkTitle: "CI/CD"
weight: 3500 
description:
---

## Overview

Previously users can use OpenFunction to build function or application source code into container images and then deploy the built image directly to the underlying sync/async Serverless runtime without user intervention. 

But OpenFunction can neither rebuild the image and then redeploy it whenever the function or application source code changes nor redeploy the image whenever this image changes (When the image is built and pushed manually or in another function)

Starting from v1.0.0, OpenFunction adds the ability to detect source code or image changes and then rebuilt and/or redeploy the new built image in a new component called `Revision Controller`. The Revision Controller is able to: 
- Detect source code changes in github, gitlab or gitee, then rebuild and redeploy the new built image whenever the source code changes.
- Detect the bundle container image (image containing the source code) changes, then rebuild and redeploy the new built image whenever the bundle image changes.
- Detect the function or application image changes, then redeploy the new image whenever the function or application image changes.

## Quick start

### Install `Revision Controller`

You can enable `Revision Controller` when installing [OpenFunction](https://openfunction.dev/docs/getting-started/installation/#install-openfunction) by simply adding the following flag to the helm command.

```shell
--set revisionController.enable=true
```

You can also enable `Revision Controller` after `OpenFunction` is installed:

```shell
kubectl apply -f https://raw.githubusercontent.com/OpenFunction/revision-controller/release-1.0/deploy/bundle.yaml
```

> The `Revision Controller` will be installed to the `openfunction` namespace by default. You can download `bundle.yaml` and change the namespace manually if you want to install it to another namespace.

### Detect source code or image changes

To detect source code or image changes, you'll need to add revision controller switch and params like below to a function's annotation.

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  annotations:
    openfunction.io/revision-controller: enable
    openfunction.io/revision-controller-params: |
      type: source
      repo-type: github
      polling-interval: 1m
  name: function-http-java
  namespace: default
spec:
  build:
    ...
  serving:
    ...
```

Annotations

| Key                                            | Description                                                                                    |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **openfunction.io/revision-controller**        | Whether to enable revision controller to detect source code or image changes for this function, can be set to either `enable` or `disable`. |
| **openfunction.io/revision-controller-params** | Parameters for revision controller.                                                            |

Parameters

| Name                  | Description                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **type**              | The change type to detect including `source`, `source-image`, and `image`.                                            |
| **polling-interval**  | The interval to polling the image digest or source code head.                                                          |
| **repo-type**         | The type of the git repo including `github`, `gitlab`, and `gitee`. Default to `github`. |
| **base-url**          | The base url of the gitlab server.                                                                                 |
| **auth-type**         | The auth type of the gitlab server.                                                                                |
| **project-id**        | The project id of a gitlab repo.                                                                                   |
| **insecure-registry** | If the image registy is insecure, you should set this to true.                                                     |
