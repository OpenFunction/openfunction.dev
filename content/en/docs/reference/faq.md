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

## Q: How to install OpenFunction in an offline environment?

A: You can install and use OpenFunction in an offline environment by following steps:

### Pull the Helm Chart

Pull the helm chart in an environment that can access GitHub:

```shell
helm repo add openfunction https://openfunction.github.io/charts/
helm repo update
helm pull openfunction/openfunction
```

Then use tools like scp to copy the helm package to your offline environment, e.g.:

```shell
scp openfunction-v1.0.0-v0.5.0.tgz <username>@<your-machine-ip>:/home/<username>/
```

### Synchronize images
You need to sync these images to your private image repository:

```
# dapr
docker.io/daprio/dashboard:0.10.0
docker.io/daprio/dapr:1.8.3

# keda
openfunction/keda:2.8.1
openfunction/keda-metrics-apiserver:2.8.1

# contour
docker.io/bitnami/contour:1.21.1-debian-11-r5
docker.io/bitnami/envoy:1.22.2-debian-11-r6
docker.io/bitnami/nginx:1.21.6-debian-11-r10

# tekton-pipelines
openfunction/tektoncd-pipeline-cmd-controller:v0.37.2
openfunction/tektoncd-pipeline-cmd-kubeconfigwriter:v0.37.2
openfunction/tektoncd-pipeline-cmd-git-init:v0.37.2
openfunction/tektoncd-pipeline-cmd-entrypoint:v0.37.2
openfunction/tektoncd-pipeline-cmd-nop:v0.37.2
openfunction/tektoncd-pipeline-cmd-imagedigestexporter:v0.37.2
openfunction/tektoncd-pipeline-cmd-pullrequest-init:v0.37.2
openfunction/tektoncd-pipeline-cmd-workingdirinit:v0.37.2
openfunction/cloudsdktool-cloud-sdk@sha256:27b2c22bf259d9bc1a291e99c63791ba0c27a04d2db0a43241ba0f1f20f4067f
openfunction/distroless-base@sha256:b16b57be9160a122ef048333c68ba205ae4fe1a7b7cc6a5b289956292ebf45cc
openfunction/tektoncd-pipeline-cmd-webhook:v0.37.2

# knative-serving
openfunction/knative.dev-serving-cmd-activator:v1.3.2
openfunction/knative.dev-serving-cmd-autoscaler:v1.3.2
openfunction/knative.dev-serving-cmd-queue:v1.3.2
openfunction/knative.dev-serving-cmd-controller:v1.3.2
openfunction/knative.dev-serving-cmd-domain-mapping:v1.3.2
openfunction/knative.dev-serving-cmd-domain-mapping-webhook:v1.3.2
openfunction/knative.dev-net-contour-cmd-controller:v1.3.0
openfunction/knative.dev-serving-cmd-default-domain:v1.3.2
openfunction/knative.dev-serving-cmd-webhook:v1.3.2

# shipwright-build
openfunction/shipwright-shipwright-build-controller:v0.10.0
openfunction/shipwright-io-build-git:v0.10.0
openfunction/shipwright-mutate-image:v0.10.0
openfunction/shipwright-bundle:v0.10.0
openfunction/shipwright-waiter:v0.10.0
openfunction/buildah:v1.23.3
openfunction/buildah:v1.28.0

# openfunction
openfunction/openfunction:v1.0.0
openfunction/kube-rbac-proxy:v0.8.0
openfunction/eventsource-handler:v4
openfunction/trigger-handler:v4
openfunction/dapr-proxy:v0.1.1
openfunction/revision-controller:v1.0.0
```

### Create custom values

Create `custom-values.yaml` in your offline environment:

```shell
touch custom-values.yaml
```

Edit `custom-values.yaml`, add the following content:

```yaml
knative-serving:
  activator:
    activator:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-activator
  autoscaler:
    autoscaler:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-autoscaler
  configDeployment:
    queueSidecarImage:
      repository: <your-private-image-repository>/knative.dev-serving-cmd-queue
  controller:
    controller:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-controller
  domainMapping:
    domainMapping:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-domain-mapping
  domainmappingWebhook:
    domainmappingWebhook:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-domain-mapping-webhook
  netContourController:
    controller:
      image:
        repository: <your-private-image-repository>/knative.dev-net-contour-cmd-controller
  defaultDomain:
    job:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-default-domain
  webhook:
    webhook:
      image:
        repository: <your-private-image-repository>/knative.dev-serving-cmd-webhook
shipwright-build:
  shipwrightBuildController:
    shipwrightBuild:
      image:
        repository: <your-private-image-repository>/shipwright-shipwright-build-controller
      GIT_CONTAINER_IMAGE:
        repository: <your-private-image-repository>/shipwright-io-build-git
      MUTATE_IMAGE_CONTAINER_IMAGE:
        repository: <your-private-image-repository>/shipwright-mutate-image
      BUNDLE_CONTAINER_IMAGE:
        repository: <your-private-image-repository>/shipwright-bundle
      WAITER_CONTAINER_IMAGE:
        repository: <your-private-image-repository>/shipwright-waiter
tekton-pipelines:
  controller:
    tektonPipelinesController:
      image:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-controller
      kubeconfigWriterImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-kubeconfigwriter
      gitImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-git-init
      entrypointImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-entrypoint
      nopImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-nop
      imagedigestExporterImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-imagedigestexporter
      prImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-pullrequest-init
      workingdirinitImage:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-workingdirinit
      gsutilImage:
        repository: <your-private-image-repository>/cloudsdktool-cloud-sdk
        digest: sha256:27b2c22bf259d9bc1a291e99c63791ba0c27a04d2db0a43241ba0f1f20f4067f
      shellImage:
        repository: <your-private-image-repository>/distroless-base
        digest: sha256:b16b57be9160a122ef048333c68ba205ae4fe1a7b7cc6a5b289956292ebf45cc
  webhook:
    webhook:
      image:
        repository: <your-private-image-repository>/tektoncd-pipeline-cmd-webhook
keda:
  image:
    keda:
      repository: <your-private-image-repository>/keda
      tag: 2.8.1
    metricsApiServer:
      repository: <your-private-image-repository>/keda-metrics-apiserver
      tag: 2.8.1
dapr:
  registry: <your-private-image-registry>/daprio
  tag: '1.8.3'
contour:
  contour:
    image:
      registry: <your-private-image-registry>
      repository: <your-private-image-repository>/contour
      tag: 1.21.1-debian-11-r5
  envoy:
    image:
      registry: <your-private-image-registry>
      repository: <your-private-image-repository>/envoy
      tag: 1.22.2-debian-11-r6
  defaultBackend:
    image:
      registry: <your-private-image-registry>
      repository: <your-private-image-repository>/nginx
      tag: 1.21.6-debian-11-r10
```

### Install OpenFunction

Run the following command in an offline environment to try to install OpenFunction：
```shell
kubectl create namespace openfunction
helm install openfunction openfunction-v1.0.0-v0.5.0.tgz -n openfunction -f custom-values.yaml
```

{{% alert title="Note" color="success" %}}

If the `helm install` command gets stuck, it may be caused by the job `contour-contour-cergen`.

Run the following command to confirm whether the job is executed successfully:

```shell
kubectl get job contour-contour-cergen -n projectcontour
```

If the job exists and the job status is completed, run the following command to complete the installation:

```shell
helm uninstall openfunction -n openfunction --no-hooks
helm install openfunction openfunction-v1.0.0-v0.5.0.tgz -n openfunction -f custom-values.yaml --no-hooks
```

{{% /alert %}}

### Patch ClusterBuildStrategy

If you want to build wasm functions or use `buildah` to build functions in offline environment, run the following command to patch `ClusterBuildStrategy`:

```shell
kubectl patch clusterbuildstrategy buildah --type='json' -p='[{"op": "replace", "path": "/spec/buildSteps/0/image", "value":"openfunction/buildah:v1.28.0"}]'
kubectl patch clusterbuildstrategy wasmedge --type='json' -p='[{"op": "replace", "path": "/spec/buildSteps/0/image", "value":"openfunction/buildah:v1.28.0"}]'
```

## Q: How to build and run functions in an offline environment

A: Let's take [Java functions](https://github.com/OpenFunction/samples/tree/main/functions/knative/java) as an example to illustrate how to build and run functions in an offline environment:

- Synchronize `https://github.com/OpenFunction/samples.git` to your private code repository

- Follow this [prerequisites](https://openfunction.dev/docs/getting-started/quickstarts/prerequisites/) doc to create `push-secret` and `git-repo-secret`

- Change public maven repository to private maven repository:
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>dev.openfunction.samples</groupId>
      <artifactId>samples</artifactId>
      <version>1.0-SNAPSHOT</version>
  
      <properties>
          <maven.compiler.source>11</maven.compiler.source>
          <maven.compiler.target>11</maven.compiler.target>
      </properties>
  
      <repositories>
          <repository>
              <id>snapshots</id>
              <name>Maven snapshots</name>
                  <!--<url>https://s01.oss.sonatype.org/content/repositories/snapshots/</url>-->
                  <url>your private maven repository</url>
              <releases>
                  <enabled>false</enabled>
              </releases>
              <snapshots>
                  <enabled>true</enabled>
              </snapshots>
          </repository>
      </repositories>
  
      <dependencies>
          <dependency>
              <groupId>dev.openfunction.functions</groupId>
              <artifactId>functions-framework-api</artifactId>
              <version>1.0.0-SNAPSHOT</version>
          </dependency>
      </dependencies>
  
  </project>
  ```
  > Make sure to commit the changes to the code repo.

- Synchronize `openfunction/buildpacks-java18-run:v1` to your private image repository

- Modify `functions/knative/java/hello-world/function-sample.yaml` according to your environment:
  ```yaml
  apiVersion: core.openfunction.io/v1beta1
  kind: Function
  metadata:
    name: function-http-java
  spec:
    version: "v2.0.0"
    image: "<your private image repository>/sample-java-func:v1"
    imageCredentials:
      name: push-secret
    port: 8080 # default to 8080
    build:
      builder: <your private image repository>/builder-java:v2-18
      params:
        RUN_IMAGE: "<your private image repository>/buildpacks-java18-run:v1"
      env:
        FUNC_NAME: "dev.openfunction.samples.HttpFunctionImpl"
        FUNC_CLEAR_SOURCE: "true"
      srcRepo:
        url: "https://<your private code repository>/OpenFunction/samples.git"
        sourceSubPath: "functions/knative/java"
        revision: "main"
        credentials:
         name: git-repo-secret
    serving:
      template:
        containers:
          - name: function # DO NOT change this
            imagePullPolicy: IfNotPresent 
      runtime: "knative"
  ```

  > If your private mirror repository is insecure, please refer to [Use private image repository in an insecure way](https://openfunction.dev/docs/reference/faq/#q-how-to-use-private-image-repositories-in-openfunction)

- Run the following commands to build and run the function:

  ```shell
  kubectl apply -f functions/knative/java/hello-world/function-sample.yaml
  ```
  
  