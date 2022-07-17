---
title: "Use SkyWalking for OpenFunction as an Observability Solution"
linkTitle: "Use SkyWalking for OpenFunction as an Observability Solution"
weight: 5300
description: >	
    Learn how to use SkyWalking for OpenFunction as an observability solution.
---

This document describes how to use SkyWalking for OpenFunction as an observability solution.

## Overview

Although FaaS allows developers to focus on their business code without worrying about the underlying implementations, it is difficult to troubleshoot the service system. OpenFunction tries to introduce capabilities of observability to improve its usability and stability.

SkyWalking provides solutions for observing and monitoring distributed systems in many different scenarios. OpenFunction has bundled go2sky(SkyWalking's Golang agent) in OpenFunction tracer options to provide distributed tracing, statistics of function performance, and functions dependency map.

## Prerequisites

- You have [installed OpenFunction](../../getting-started/installation/).
- You have [created a secret](../../getting-started/your-first-function/function-go/#create-a-secret).
- You have [installed SkyWalking v9](https://github.com/apache/skywalking-kubernetes#apache-skywalking-kubernetes).
- You have [created a Kafka server and topic](../interact-with-dapr-output-binding/#create-a-kafka-server-and-topic).

## Tracing Parameters

The following table describes the tracing parameters.

| Name               | Description                                                  | Example                |
| ------------------ | ------------------------------------------------------------ | ---------------------- |
| enabled            | Switch for tracing, default to `false`.                      | true, false            |
| provider.name      | Provider name can be set to "skywalking", "opentelemetry" (pending). | "skywalking"           |
| provider.oapServer | The oap server address.                                      | "skywalking-opa:11800" |
| tags               | A collection of key-value pairs for Span custom tags in tracing. |                        |
| tags.func          | The name of function. It will be automatically filled.       | "function-a"           |
| tags.layer         | Indicates the type of service being tracked. It should be set to "faas" when you use the function. | "faas"                 |
| baggage            | A collection of key-value pairs, exists in the tracing and also needs to be transferred across process boundaries. |                        |

The following is a JSON formatted configuration reference that guides the formatting structure of the tracing configuration.

```json
{
  "enabled": true,
  "provider": {
    "name": "skywalking",
    "oapServer": "skywalking-oap:11800"
  },
  "tags": {
    "func": "function-a",
    "layer": "faas",
    "tag1": "value1",
    "tag2": "value2"
  },
  "baggage": {
    "key": "key1",
    "value": "value1"
  }
}
```

## Enable Tracing Configuration of OpenFunction

### Option 1: global configuration

This document uses `skywalking-oap.default:11800` as an example of the skywalking-oap address in the cluster.

1. Run the following command to modify the configmap `openfunction-config` in the `openfunction` namespace.

   ```shell
   kubectl edit configmap openfunction-config -n openfunction
   ```

2. Modify the content under `data.plugins.tracing` by referring to the following example and save the change.

   ```yaml
   data:
     plugins.tracing: |
       enabled: true
       provider:
         name: "skywalking"
         oapServer: "skywalking-oap:11800"
       tags:
         func: tracing-function
         layer: faas
         tag1: value1
         tag2: value2
       baggage:
         key: "key1"
         value: "value1"
   ```

### Option 2: function-level configuration

To enable tracing configuration in the function-level, add the field `plugins.tracing` under `metadata.annotations` in the function manifest as the following example.

```yaml
metadata:
  name: tracing-function
  annotations:
    plugins.tracing: |
      enabled: true
      provider:
        name: "skywalking"
        oapServer: "skywalking-oap:11800"
      tags:
        func: tracing-function
        layer: faas
        tag1: value1
        tag2: value2
      baggage:
        key: "key1"
        value: "value1"
```

It is recommended that you use the global tracing configuration, or you have to add function-level tracing configuration for every function you create.

## Use SkyWalking as a Distributed Tracing Solution

1. Create functions by referring to [this document](../interact-with-dapr-output-binding/).

2. Then, you can observe the flow of entire link on the SkyWalking UI.

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-topology.gif)

3. You can also observe the comparison of the response time of the Knative runtime function (function-front) in the running state and under cold start.

   In cold start:

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-time-in-cold-start.png)

   In running:

   ![](/images/docs/en/best-practices/skywalking-solution-for-openfunction/tracing-time-in-running.png)

