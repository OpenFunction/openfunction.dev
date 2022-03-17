---
title: "Your First Function: Go"
linkTitle: "Your First Function: Go"
weight: 2210
description: >	
    Learn how to create a function in Go.
---

This document describes how to create a function in Go.

## Prerequisites

You have [installed OpenFunciton](../..//installation/).

## Create a Function in Go

### Create a secret

Define the values of `REGISTRY_SERVER`, `REGISTRY_USER` and `REGISTRY_PASSWORD` in the following command, and then run it to create a secret for accessing your image registry.

```shell
REGISTRY_SERVER=https://index.docker.io/v1/ REGISTRY_USER=<your registry username> REGISTRY_PASSWORD=<your registry password>
kubectl create secret docker-registry push-secret \
    --docker-server=$REGISTRY_SERVER \
    --docker-username=$REGISTRY_USER \
    --docker-password=$REGISTRY_PASSWORD
```

### Create a function in Go

1. Use the following example YAML file to create a manifest `function-sample.yaml` for your function, and modify the value of `spec.image` to set your own image registry address.

   ```yaml
   apiVersion: core.openfunction.io/v1beta1
   kind: Function
   metadata:
     name: function-sample
   spec:
     version: "v2.0.0"
     image: "<your registry name>/sample-go-func:v2"
     imageCredentials:
       name: push-secret
     port: 8080 # Default to 8080.
     build:
       builder: openfunction/builder-go:v2-1.16
       env:
         FUNC_NAME: "HelloWorld"
         FUNC_CLEAR_SOURCE: "true"
       srcRepo:
         url: "https://github.com/OpenFunction/samples.git"
         sourceSubPath: "functions/Knative/hello-world-go"
         revision: "release-0.6"
     serving:
       template:
         containers:
           - name: function
             imagePullPolicy: IfNotPresent
       runtime: "knative"
   ```
   
2. Run the following command to create the function.

   ```shell
   kubectl apply -f function-sample.yaml
   ```

### Access the function

#### Option 1:

1. Run the following command to view the status of the function. The `URL` in the output is the address provided by the OpenFunction Domain for accessing the function. Before accessing the function through the URL, you need to make sure that it can be resolved by DNS. 

   ```shell
   $ kubectl get functions.core.openfunction.io
   
   NAME              BUILDSTATE   SERVINGSTATE   BUILDER         SERVING         URL                                              AGE
   function-sample   Succeeded    Running        builder-jgnzp   serving-q6wdp   http://openfunction.io/default/function-sample   22m
   ```

2. Run the following command to create a pod in the cluster for accessing the function from the pod.

   ```shell
   kubectl run curl --image=radial/busyboxplus:curl -i --tty
   ```
   
2. Run the following commands in the pod to access the function through the URL.

   ```shell
   [ root@curl:/ ]$ curl http://openfunction.io.svc.cluster.local/default/function-sample/World
   Hello, World!
   
   [ root@curl:/ ]$ curl http://openfunction.io.svc.cluster.local/default/function-sample/OpenFunction
   Hello, OpenFunction!
   ```

#### Option 2:

You can also access the function through the URL provided by the Knative service.

1. As Kourier is the network layer of Knative, you can simply use your node IP address as the `<external-ip>` to indicate the external IP address of your gateway service. Make sure you replace `<external-ip>` with your node IP address.

   ```shell
   kubectl patch svc -n kourier-system kourier \
     -p '{"spec": {"type": "LoadBalancer", "externalIPs": ["<external-ip>"]}}'
   
   kubectl patch configmap/config-domain -n knative-serving \
     --type merge --patch '{"data":{"<external-ip>.sslip.io":""}}'
   ```

2. Run the following command to obtain the service entry of the function.

   ```shell
   $ kubectl get ksvc
    
   NAME                       URL                                                              LATESTCREATED                   LATESTREADY                     READY   REASON
   serving-q6wdp-ksvc-wk6mv   http://serving-q6wdp-ksvc-wk6mv.default.<external-ip>.sslip.io   serving-q6wdp-ksvc-wk6mv-v100   serving-q6wdp-ksvc-wk6mv-v100   True
   ```

3. (Optional) You can also run the following command to obtain the service entry of the function.

   ```shell
   $ kubectl get ksvc serving-q6wdp-ksvc-wk6mv -o jsonpath={.status.url}
    
   http://serving-q6wdp-ksvc-wk6mv.default.<external-ip>.sslip.io
   ```

4. Run the following commands to access the function through the Knative service.

   ```shell
   $ curl http://serving-q6wdp-ksvc-wk6mv.default.<external-ip>.sslip.io/World
    
   Hello, World!
   
   $ curl http://serving-q6wdp-ksvc-wk6mv.default.<external-ip>.sslip.io/OpenFunction
    
   Hello, OpenFunction!
   ```



