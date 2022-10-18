---
title: "Function Scaling and Triggers"
linkTitle: "Function Scaling and Triggers"
weight: 3210
description: 
---

Scaling is one of the core features of a FaaS or Serverless platform.

OpenFunction defines function scaling in `ScaleOptions` and defines triggers to activate function scaling in `Triggers`

## ScaleOptions

You can define unified function scale options for sync and async functions like below which will be valid for both sync and async functions:

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      minReplicas: 0
      maxReplicas: 10
```

Usually simply defining `minReplicas` and `maxReplicas` is not enough for async functions. You can define seperate scale options for async functions like below which will override the `minReplicas` and `maxReplicas`.

> You can find more details of async function scale options in [KEDA ScaleObject Spec](https://keda.sh/docs/2.7/concepts/scaling-deployments/#scaledobject-spec) and [KEDA ScaledJob Spec](https://keda.sh/docs/2.7/concepts/scaling-jobs/#scaledjob-spec).

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      keda:
        scaledObject:
          pollingInterval: 15
          minReplicaCount: 0
          maxReplicaCount: 10
          cooldownPeriod: 60
          advanced:
            horizontalPodAutoscalerConfig:
              behavior:
                scaleDown:
                  stabilizationWindowSeconds: 45
                  policies:
                  - type: Percent
                    value: 50
                    periodSeconds: 15
                scaleUp:
                  stabilizationWindowSeconds: 0
```

You can also set advanced scale options for Knative sync functions too which will override the `minReplicas` and `maxReplicas`.

> You can find more details of the Knative sync function scale options [here](https://knative.dev/docs/serving/autoscaling/scale-bounds/#scale-down-delay)

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
        autoscaling.knative.dev/min-scale: "0"
        autoscaling.knative.dev/max-scale: "10"
        autoscaling.knative.dev/initial-scale: "1"
        autoscaling.knative.dev/scale-down-delay: "0"
        autoscaling.knative.dev/window: "60s"
        autoscaling.knative.dev/panic-window-percentage: "10.0"
        autoscaling.knative.dev/metric: "concurrency"
        autoscaling.knative.dev/target: "100"
```

## Triggers

Triggers define how to activate function scaling for async functions. You can use triggers defined in all [KEDA scalers](https://keda.sh/docs/2.7/scalers/) as OpenFunction's trigger spec.

> Sync functions' scaling is activated by various options of HTTP requests which are already defined in the previous ScaleOption section.

```yaml
apiVersion: core.openfunction.io/v1beta1
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    runtime: "async"
    triggers:
      - type: kafka
        metadata:
          topic: logs
          bootstrapServers: kafka-server-kafka-brokers.default.svc.cluster.local:9092
          consumerGroup: logs-handler
          lagThreshold: "20"
```