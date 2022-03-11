---
title: "Your First Function: Node.js"
linkTitle: "Your First Function: Node.js"
weight: 2230
description: >	
    Learn how to create a function in Node.js.
---

This document described how to create a function in Node.js.

## Prerequisites

You have [installed OpenFunciton](../../installation/).

## Create a Function in Node.js

### Create a secret

Define the values of `REGISTRY_SERVER`, `REGISTRY_USER` and `REGISTRY_PASSWORD` in the following command, and then run it to create a secret for accessing your image registry.

```shell
REGISTRY_SERVER=https://index.docker.io/v1/ REGISTRY_USER=<your_registry_username> REGISTRY_PASSWORD=<your_registry_password>
kubectl create secret docker-registry push-secret \
    --docker-server=$REGISTRY_SERVER \
    --docker-username=$REGISTRY_USER \
    --docker-password=$REGISTRY_PASSWORD
```

### Create a function in Node.js

1. Use the following example YAML file to create a manifest `function-sample.yaml` for your function, and modify the value of `spec.image` to set your own image registry address.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: node-sample
   spec:
     version: "v1.0.0"
     image: "<your registry name>/sample-node-func:v2"
     imageCredentials:
       name: push-secret
     port: 8080 # Default to 8080.
     build:
       builder: "openfunction/builder-node:v2-16.13"
       env:
         FUNC_NAME: "helloWorld"
         FUNC_TYPE: "http"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/Knative/hello-world-node/commonjs"
         revision: "release-0.6"
     serving:
       runtime: "knative" # Default to knative.
       template:
         containers:
           - name: function
             imagePullPolicy: IfNotPresent
   ```

2. Run the following command to create the function.

   ```shell
   kubectl apply -f function-sample.yaml
   ```

### Access the function

For more information about how to access the function, refer to [Access the function](../function-go#access-the-function).
