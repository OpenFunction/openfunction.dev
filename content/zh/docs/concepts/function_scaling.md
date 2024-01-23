---
title: "函数缩放"
linkTitle: "函数缩放"
weight: 3220
description:
---

缩放是 FaaS 或 Serverless 平台的核心特性之一。

OpenFunction 在 `ScaleOptions` 中定义了函数缩放，并在 `Triggers` 中定义了激活函数缩放的触发器。

## ScaleOptions

您可以为同步和异步函数定义统一的函数缩放选项，如下所示，这将对同步和异步函数都有效：

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      minReplicas: 0
      maxReplicas: 10
```

通常，仅定义 minReplicas 和 maxReplicas 对于异步函数来说是不够的。您可以像下面这样为异步函数定义单独的缩放选项，这将覆盖 minReplicas 和 maxReplicas。

您可以在 [KEDA ScaleObject Spec](https://keda.sh/docs/2.7/concepts/scaling-deployments/#scaledobject-spec) 和 [KEDA ScaledJob Spec](https://keda.sh/docs/2.7/concepts/scaling-jobs/#scaledjob-spec) 中找到更多关于异步函数缩放选项的详细信息。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      minReplicas: 0
      maxReplicas: 10
      keda:
        scaledObject:
          pollingInterval: 15
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

您也可以为 Knative 同步函数设置高级缩放选项，这将覆盖 minReplicas 和 maxReplicas。

> 您可以在 [这里](https://knative.dev/docs/serving/autoscaling/scale-bounds/#scale-down-delay) 找到更多关于 Knative 同步函数缩放选项的详细信息。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      knative:
        autoscaling.knative.dev/initial-scale: "1"
        autoscaling.knative.dev/scale-down-delay: "0"
        autoscaling.knative.dev/window: "60s"
        autoscaling.knative.dev/panic-window-percentage: "10.0"
        autoscaling.knative.dev/metric: "concurrency"
        autoscaling.knative.dev/target: "100"
```

### 触发器

触发器定义了如何激活异步函数的函数缩放。您可以使用所有 [KEDA 缩放器](https://keda.sh/docs/2.7/scalers/) 中定义的触发器作为 OpenFunction 的触发器规范。

> 同步函数的缩放是通过 HTTP 请求的各种选项激活的，这些选项已经在前面的 ScaleOption 部分中定义了。

```yaml
apiVersion: core.openfunction.io/v1beta2
kind: Function
metadata:
  name: function-sample
spec:
  serving:
    scaleOptions:
      keda:
        triggers:
          - type: kafka
            metadata:
              topic: logs
              bootstrapServers: kafka-server-kafka-brokers.default.svc.cluster.local:9092
              consumerGroup: logs-handler
              lagThreshold: "20"
```
