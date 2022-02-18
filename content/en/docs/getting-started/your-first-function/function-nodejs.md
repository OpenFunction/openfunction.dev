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
   apiVersion: core.openfunction.io/v1alpha2
   kind: Function
   metadata:
     name: node-sample
   spec:
     version: "v1.0.0"
     image: "<your_registry_username>/sample-node-func:v0.5"
     imageCredentials:
       name: push-secret
     port: 8080 # Defaults to 8080.
     build:
       builder: "openfunction/gcp-builder:v1"
       env:
         GOOGLE_FUNCTION_TARGET: "helloWorld"
         GOOGLE_FUNCTION_SIGNATURE_TYPE: "http"
       srcRepo:
         revision: "release-0.5"
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/Knative/hello-world-node"
     serving:
       runtime: "Knative" # Defaults to Knative.
   ```

2. Run the following command to create the function.

   ```shell
   kubectl apply -f function-sample.yaml
   ```

### Check the result

For more information about how to check the result, refer to [Check the result](../function-go#check-the-result).
