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
   apiVersion: core.openfunction.io/v1alpha2
   kind: Function
   metadata:
     name: function-sample
   spec:
     image: "<your registry name>/sample-go-func:latest"
   ```
   
2. Run the following command to create the function.

   ```shell
   kubectl apply -f function-sample.yaml
   ```

### Check the result

1. Run the following command to view the function.

   ```shell
   $ kubectl get functions.core.openfunction.io
   
   NAME              AGE
   function-sample   5s
   ```

2. Run the following command to view the final state of the function workload in the Serving.

   ```shell
   $ kubectl get servings.core.openfunction.io
    
   NAME                      AGE
   function-sample-serving   15s
   ```

3. Run the following command to obtain the service entry of the function.

   ```shell
   $ kubectl get ksvc
    
   NAME                           URL                                                                  LATESTCREATED                        LATESTREADY                          READY   REASON
   function-sample-serving-ksvc   http://function-sample-serving-ksvc.default.<external-ip>.sslip.io   function-sample-serving-ksvc-00001   function-sample-serving-ksvc-00001   True
   ```

4. (Optional) You can also run the following command to obtain the service entry of the function.

   ```shell
   $ kubectl get ksvc function-sample-serving-ksvc -o jsonpath={.status.url}
    
   http://function-sample-serving-ksvc.default.<external-ip>.sslip.io
   ```

5. (Optional) The `<external-ip>` parameter indicates the external IP address of your gateway service. If you use Kourier as the network layer of Knative, run the following commands to use your node IP address. Make sure you replace `<your-node-IP>` with your node IP address.

   ```shell
   kubectl patch svc -n kourier-system kourier \
     -p '{"spec": {"type": "LoadBalancer", "externalIPs": ["<your node IP>"]}}'
   
   kubectl patch configmap/config-domain -n knative-serving \
     --type merge --patch '{"data":{"1.2.3.4.sslip.io":""}}'
   ```

6. Run the following command to access the service.

   ```shell
   $ curl http://function-sample-serving-ksvc.default.<external-ip>.sslip.io
    
   Hello, World!
   ```



