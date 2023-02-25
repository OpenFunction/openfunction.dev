---
title: "Serverless Applications"
linkTitle: "Serverless Applications"
weight: 3330
description: 
---
In addition to building and running Serverless Functions, you can also build and run Serverless Applications with OpenFuntion. 

OpenFunction support building source code into container images in two different ways: 
- Using [Cloud Native Buildpacks](https://buildpacks.io/) to build source code without a `Dockerfile`
- Using [Buildah](https://buildah.io/) or [BuildKit](https://github.com/moby/buildkit) to build source code with a `Dockerfile`

> To push images to a container registry, you'll need to create a secret containing the registry's credential and add the secret to `imageCredentials`.
> Please refer to the [prerequisites](../../getting-started/Quickstarts/prerequisites) section for more info.

## Build and run a Serverless Application with a Dockerfile

If you already created a `Dockerfile` for your application like this [Go Application](https://github.com/OpenFunction/samples/tree/main/apps/buildah/go), you can build and run this application in the serverless way like [this](https://github.com/OpenFunction/samples/blob/main/apps/buildah/go/sample-go-app.yaml):

1. Create the sample go serverless application

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: sample-go-app
spec:
  version: "v1.0.0"
  image: "openfunctiondev/sample-go-app:v1"
  imageCredentials:
    name: push-secret
  #port: 8080 # default to 8080
  build:
    builder: openfunction/buildah:v1.23.1
    srcRepo:
      url: "https://github.com/OpenFunction/samples.git"
      sourceSubPath: "apps/buildah/go"
      revision: "main"
    shipwright:
      strategy:
        name: buildah
        kind: ClusterBuildStrategy
  serving:
    runtime: knative
    template:
      containers:
        - name: function
          imagePullPolicy: IfNotPresent
EOF
```

2. Check the application status

You can then check the serverless app's status by `kubectl get functions.core.openfunction.io -w`:

```shell
kubectl get functions.core.openfunction.io -w
NAME                    BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                   AGE
sample-go-app           Succeeded    Running        builder-jgnzp   serving-q6wdp   http://sample-go-app.default.svc.cluster.local/           22m
```

3. Access this application

Once the `BUILDSTATE` becomes `Succeeded` and the `SERVINGSTATE` becomes `Running`, you can access this Go serverless app through the address in the `ADDRESS` field: 

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://sample-go-app.default.svc.cluster.local
```

> [Here](https://github.com/OpenFunction/samples/tree/main/apps/buildah/java) you can find a Java Serverless Applications (with a Dockerfile) example.

## Build and run a Serverless Application without a Dockerfile

If you hava an application without a `Dockerfile` like this [Java Application](https://github.com/buildpacks/samples/tree/main/apps/java-maven), you can also build and run your application in the serverless way like this [Java application](https://github.com/OpenFunction/samples/tree/main/apps/buildpacks/java):

1. Create the sample Java serverless application

```shell
cat <<EOF | kubectl apply -f -
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: sample-java-app-buildpacks
spec:
  version: "v1.0.0"
  image: "openfunction/sample-java-app-buildpacks:v1"
  imageCredentials:
    name: push-secret
  port: 8080 # default to 8080
  build:
    builder: "cnbs/sample-builder:alpine"
    srcRepo:
      url: "https://github.com/buildpacks/samples.git"
      sourceSubPath: "apps/java-maven"
      revision: "main"
  serving:
    runtime: "knative" # default to knative
    template:
      containers:
        - name: function
          imagePullPolicy: IfNotPresent
EOF
```

1. Check the application status

You can then check the serverless app's status by `kubectl get functions.core.openfunction.io -w`:

```shell
kubectl get functions.core.openfunction.io -w
NAME                                 BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         ADDRESS                                                                AGE
sample-java-app-buildpacks           Succeeded    Running        builder-jgnzp   serving-q6wdp   http://sample-java-app-buildpacks.default.svc.cluster.local/           22m
```

1. Access this application

Once the `BUILDSTATE` becomes `Succeeded` and the `SERVINGSTATE` becomes `Running`, you can access this Java serverless app through the address in the `ADDRESS` field: 

```shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
curl http://sample-java-app-buildpacks.default.svc.cluster.local
```