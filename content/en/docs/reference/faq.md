---
title: "FAQ"
linkTitle: "FAQ"
weight: 7100
description: 
---

This document describes FAQs when using OpenFunction.

## Q: How to use private image repositories in OpenFunction?

A: OpenFunction uses Shipwright (which utilizes Tekton to integrate with Cloud Native Buildpacks) in the build phase to package the user function to the application image.

Users often choose to access a private image repository in an insecure way, which is not yet supported by the Cloud Native Buildpacks.

We offer a workaround as below to get around this limitation for now:

1. Use IP address instead of hostname as access address for private image repository.
2. You should [skip tag-resolution](https://knative.dev/docs/serving/configuration/deployment/#skipping-tag-resolution) when you run the Knative-runtime function.

For references:

[buildpacks/lifecycle#524](https://github.com/buildpacks/lifecycle/issues/524)

[buildpacks/tekton-integration#31](https://github.com/buildpacks/tekton-integration/issues/31)

## Q: How to access the Knative-runtime function without introducing a new ingress controller?

A: OpenFunction provides a unified entry point for function accessibility, which is based on the Ingress Nginx implementation. However, for some users, this is not necessary, and instead, introducing a new ingress controller may affect the current cluster.

In general, accessible addresses are for the sync(Knative-runtime) functions. Here are two ways to solve this problem:

- Magic DNS

  You can follow [this guide](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#configure-dns) to config the DNS.

- CoreDNS

  This is similar to using Magic DNS, with the difference that the configuration for DNS resolution is placed inside CoreDNS. Assume that the user has configured a domain named "openfunction.dev" in the ConfigMap `config-domain` under the `knative-serving` namespace (as shown below):

  ```shell
  $ kubectl -n knative-serving get cm config-domain -o yaml
  
  apiVersion: v1
  data:
    openfunction.dev: ""
  kind: ConfigMap
  metadata:
    annotations:
      knative.dev/example-checksum: 81552d0b
    labels:
      app.kubernetes.io/part-of: knative-serving
      app.kubernetes.io/version: 1.0.1
      serving.knative.dev/release: v1.0.1
    name: config-domain
    namespace: knative-serving
  ```

  Next, let's add an A record for this domain. OpenFunction uses Kourier as the default network layer for Knative Serving, which is where the domain should flow to.

  ```shell
  $ kubectl -n kourier-system get svc
  
  NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
  kourier            LoadBalancer   10.233.7.202   <pending>     80:31655/TCP,443:30980/TCP   36m
  kourier-internal   ClusterIP      10.233.47.71   <none>        80/TCP                       36m
  ```

  Then the user only needs to configure this Wild-Card DNS resolution in CoreDNS to resolve the URL address of any Knative Service in the cluster.

  > Where "10.233.47.71" is the address of the Service kourier-internal.

  ```shell
  $ kubectl -n kube-system get cm coredns -o yaml
  
  apiVersion: v1
  data:
    Corefile: |
      .:53 {
          errors
          health
          ready
          template IN A openfunction.dev {
            match .*\.openfunction\.dev
            answer "{{ .Name }} 60 IN A 10.233.47.71"
            fallthrough
          }
          kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
          }
          hosts /etc/coredns/NodeHosts {
            ttl 60
            reload 15s
            fallthrough
          }
          prometheus :9153
          forward . /etc/resolv.conf
          cache 30
          loop
          reload
          loadbalance
      }
      ...
  ```

  If the user cannot resolve the URL address for this function outside the cluster, configure the `hosts` file as follows:

  > Where "serving-sr5v2-ksvc-sbtgr.default.openfunction.dev" is the URL address obtained from the command "kubectl get ksvc".

  ```shell
  10.233.47.71 serving-sr5v2-ksvc-sbtgr.default.openfunction.dev
  ```

After the above configuration is done, you can get the URL address of the function with the following command. Then you can trigger the function via `curl` or your browser.

```shell
$ kubectl get ksvc

NAME                       URL
serving-sr5v2-ksvc-sbtgr   http://serving-sr5v2-ksvc-sbtgr.default.openfunction.dev
```

## Q: How to enable and configure concurrency for functions?

A: OpenFunction categorizes function types into "[sync runtime](../../concepts/function/#the-sync-runtime)" and "[async runtime](../../concepts/function/#the-async-runtime)" based on the type of request being handled. These two types of functions are driven by Knative Serving and Dapr + KEDA.

Therefore, to enable and configure the concurrency of functions, you need to refer to the specific implementation in the above components. 

The following section describes how to enable and configure concurrency of functions in OpenFunction according to the "[sync runtime](../../concepts/function/#the-sync-runtime)" and "[async runtime](../../concepts/function/#the-async-runtime)" sections.

### Sync runtime

You can start by referring to this [document](https://knative.dev/docs/serving/autoscaling/concurrency/) in Knative Serving on enabling and configuring concurrency capabilities.

Knative Serving has **Soft limit** and **Hard limit** configurations for the concurrency feature.

#### Soft limit

You can refer to the `Global(ConfigMap)` and `Global(Operator)` sections of this [document](https://knative.dev/docs/serving/autoscaling/concurrency/#soft-limit) to configure global concurrency capabilities.

And for `Per Revision` you can configure it like [this](../../concepts/function_scaling_trigger/).

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
        autoscaling:
          target: "200"
```

#### Hard limit

OpenFunction currently doesn't support configuring Hard limit for `Per Revision`. You can refer to the `Global(ConfigMap)` and `Global(Operator)` sections of this [document](https://knative.dev/docs/serving/autoscaling/concurrency/#hard-limit) to configure global concurrency capabilities.

#### In summary

In a nutshell, you can configure Knative Serving's Autoscaling-related configuration items for each function as follows, just as long as they can be passed in as ***Annotation***, otherwise, you can only do global settings.

```yaml
# Configuration in Knative Serving
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/<key>: "value"

# Configuration in OpenFunction (recommended)
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
        autoscaling:
          <key>: "value"

# Alternative approach
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    annotations:
      autoscaling.knative.dev/<key>: "value"
```

### Async runtime

You can start by referring to the [document](https://docs.dapr.io/operations/configuration/control-concurrency/) in Dapr on enabling and configuring concurrency capabilities.

Compared to the concurrency configuration of sync runtime, the concurrency configuration of async runtime is simpler.

```yaml
# Configuration in Dapr
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodesubscriber
  namespace: default
spec:
  template:
    metadata:
      annotations:
        dapr.io/app-max-concurrency: "value"

# Configuration in OpenFunction (recommended)
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    annotations:
      dapr.io/app-max-concurrency: "value"
```

## Q: How to create source repository credential for the function image build process?

A: You may be prompted with errors like `Unsupported type of credentials provided, either SSH private key or username/password is supported (exit code 110)` when using `spec.build.srcRepo.credentials`, which means you are using an incorrect Secret resource as source repository crendital.

OpenFunction currently implements the function image building framework based on [ShipWright](https://shipwright.io/), so we need to refer to this [document](https://shipwright.io/docs/build/authentication/#authentication-for-git) to setup the correct source repository credential.
