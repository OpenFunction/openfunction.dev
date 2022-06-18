---
title: "Domain"
linkTitle: "Domain"
weight: 4700
description:
---

This document describes the concept of Domain in OpenFunction.

## Domain

Domain defines a unified entry point for synchronous functions through ingress. For example, you can use the following URL to access a function.

```
http://<domain name>.<domain namespace>/<function namespace>/<function name>
```

In a cluster, you can define only one `Domain`, which requires an ingress controller. By default, OpenFunction uses `nginx-ingress`. You can use the `--ingress` parameter of `ofn` to install it.

If your `nginx-ingress` does not use the default namespace and name, please modify [default-domain.yaml](https://github.com/OpenFunction/OpenFunction/blob/release-0.6/config/domain/default-domain.yaml) in your repository, and run the following command to update [bundle.yaml](https://github.com/OpenFunction/OpenFunction/blob/release-0.6/config/bundle.yaml).

```
make manifests
```

Then, apply the updated file to deploy OpenFunction.
