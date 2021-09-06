---
title: "Function 参数说明"
linkTitle: "Function 参数说明"
weight: 5
description: >
    Function CRD 参数规范说明
---

## Function

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | core.openfunction.io/v1alpha1                                |
| **kind** *string*                                            | Function                                                     |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *（可选）* 参考 [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) 文档 |
| **spec** *[FunctionSpec](#functionspec)*                     | Function 的规格，参考 [FunctionSpec](#functionspec)          |
| **status** *FunctionStatus*                                  | Function 的状态，参考 FunctionStatus                         |

### FunctionSpec

*从属 [Function](#function)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **version** *string*                                         | *（可选）* 函数版本，如：“v1.0.0”                            |
| **image** *string*                                           | 函数构建后的镜像上传路径，如：“demorepo/demofunction:v1”     |
| **imageCredentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *（可选）* 访问镜像仓库的凭证，参考 [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |
| **port** *int32*                                             | *（可选）* 函数应用监听的端口，如：“8080”                    |
| **build** *[BuildImpl](#buildimpl)*                          | *（可选）* 函数的 Builder 规格，参考 [BuildImpl](#buildimpl) |
| **serving** *[ServingImpl](#servingimpl)*                    | *（可选）* 函数的 Serving 规格，参考 [ServingImpl](#servingimpl) |

### BuildImpl

*从属 [FunctionSpec](#functionspec)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **builder** *string*                                         | 镜像构建器的名称                                             |
| **builderCredentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *（可选）* 访问镜像仓库的凭证，参考 [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |
| **shipwright** *[ShipwrightEngine](#shipwrightengine)*       | *（可选）* Shipwright 引擎的规格，参考 [ShipwrightEngine](#shipwrightengine) |
| **params** *map[string]string*                               | *（可选）* 传递给 Shipwright 的参数                          |
| **env** map[string]string                                    | *（可选）* 传递给镜像构建器的参数                            |
| **srcRepo** *[GitRepo](#gitrepo)*                            | 源代码仓库的配置，参考 [GitRepo](#gitrepo)                   |
| **dockerfile** *string*                                      | *（可选）* Dockerfile 文件的路径，用于指导 Shipwright 使用 Dockerfile 构建镜像 |

### ShipwrightEngine

*从属 [BuildImpl](#buildimpl)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **strategy** *[Strategy](#strategy)*                         | *（可选）* 镜像构建策略的名称，参考 [Strategy](#strategy)    |
| **timeout** *[v1.Duration](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#Duration)* | *（可选）* 镜像构建超时时间，参考 [v1.Duration](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#Duration) |

### Strategy

*从属 [ShipwrightEngine](#shipwrightengine)*

| 字段            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| **name** string | 镜像构建策略的名称                                           |
| **kind** string | *（可选）* 镜像构建策略的 Kind，默认为“BuildStrategy”，可选“ClusterBuildStrategy” |

### GitRepo

*从属 [BuildImpl](#buildimpl)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **url** *string*                                             | 代码仓库地址                                                 |
| **revision** *string*                                        | *（可选）* 代码仓库中的可引用实例，如 commit id，branch name 等 |
| **sourceSubPath** *string*                                   | *（可选）* 目标函数在代码仓库中的目录，如：“functions/function-a/” |
| **credentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *（可选）* 代码仓库的访问凭证，参考 [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |

### ServingImpl

*从属 [FunctionSpec](#functionspec)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **runtime** *string*                                         | 负载运行时的类型，可选：“**Knative**” 和 “**OpenFuncAsync**” |
| **params** map[string]string                                 | *（可选）* 传递给应用负载的环境变量参数                      |
| **openFuncAsync** *[OpenFuncAsyncRuntime](#openfuncasyncruntime)* | *（可选）* 当 runtime 为 OpenFuncAsync 时，用于定义 OpenFuncAsync 的配置，参考 [OpenFuncAsyncRuntime](#openfuncasyncruntime) |
| **template** *[v1.PodSpec](https://pkg.go.dev/k8s.io/api/core/v1#PodSpec)* | *（可选）* 应用负载中 Pod 的定义模板，参考 [v1.PodSpec](https://pkg.go.dev/k8s.io/api/core/v1#PodSpec) |

### OpenFuncAsyncRuntime

*从属 [ServingImpl](#servingimpl)*

| 字段                     | 描述                                                  |
| ------------------------ | ----------------------------------------------------- |
| **dapr** *[Dapr](#dapr)* | *（可选）* Dapr components 的定义，参考 [Dapr](#dapr) |
| **keda** *[Keda](#keda)* | *（可选）* Keda 的定义，参考 [Keda](#keda)            |

### Dapr

*从属 [OpenFuncAsyncRuntime](#openfuncasyncruntime)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **annotations** *map[string]string*                          | *（可选）* Dapr components 的注解，参考 [Dapr 相关文档](https://docs.dapr.io/reference/arguments-annotations-overview/) |
| **components** *\[][DaprComponent](#daprcomponent)*          | *（可选）* Dapr Components Spec 数组，参考 [DaprComponent](#daprcomponent) |
| **subscriptions** *\[][DaprSubscription](#daprsubscription)* | *（可选）* Dapr Subscription Spec 数组，参考 [DaprSubscription](#daprsubscription) |
| **inputs** *\[][DaprIO](#daprio)*                            | *（可选）* 函数输入端的定义，参考 [DaprIO](#daprio)          |
| **outputs** *\[][DaprIO](#daprio)*                           | *（可选）* 函数输出端的定义，参考 [DaprIO](#daprio)          |

### DaprComponent

*从属 [Dapr](#dapr)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **name** *string*                                            | Dapr component 的名称                                        |
| *[v1alpha1.ComponentSpec](https://pkg.go.dev/github.com/dapr/dapr/pkg/apis/components/v1alpha1#ComponentSpec)* | Dapr Components Spec 定义，参考 [Dapr 相关文档](https://docs.dapr.io/reference/components-reference/) |

### DaprSubscription

*从属 [Dapr](#dapr)*

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **name** *string*                                            | Dapr components 的注解，参考 [Dapr 相关文档](https://docs.dapr.io/reference/arguments-annotations-overview/) |
| *[v1alpha1.SubscriptionSpec](https://pkg.go.dev/github.com/dapr/dapr/pkg/apis/subscriptions/v1alpha1#SubscriptionSpec)* | Dapr Subscription Spec 定义，参考 [Dapr 相关文档](https://docs.dapr.io/reference/components-reference/) |
| **scopes** *[]string*                                        |                                                              |

### DaprIO

*从属 [Dapr](#dapr)*

| 字段                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| **name** *string*              | 函数输入、输出端的名称，与 [DaprComponent](#daprcomponent) 的 name 一致即表示关联 |
| **type** *string*              | Dapr component 的类型，可选：`bindings`、`pubsub`、`invoke`  |
| **topic** *string*             | *（可选）* 当 **type** 为 `pubsub` 时，需要设置 topic        |
| **methodName** *string*        | *（可选）* 当 **type** 为 `invoke` 时，需要设置 methodName，参考 [Dapr 相关文档](https://docs.dapr.io/reference/api/service_invocation_api/#url-parameters) |
| **params** *map[string]string* | *（可选）* 传递给 Dapr 的参数                                |

### Keda

*从属 [OpenFuncAsyncRuntime](#openfuncasyncruntime)*

| 字段                                                     | 描述                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| **scaledObject** *[KedaScaledObject](#kedascaledobject)* | KEDA 可扩展对象（Deployments）定义，参考 [KedaScaledObject](#kedascaledobject) |
| **scaledJob** *[KedaScaledJob](#kedascaledjob)*          | KEDA 可扩展任务定义，参考 [KedaScaledJob](#kedascaledjob)    |

### KedaScaledObject

*从属 [Keda](#keda)*

> 你可以参考 [Scaling Deployments, StatefulSets & Custom Resources](https://keda.sh/docs/2.4/concepts/scaling-deployments/) 获得更多信息

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **workloadType** *string*                                    | 以何种方式运行函数，可选：`Deployment`、 `StatefulSet`，默认为 `Deployment`. |
| **pollingInterval** *int32*                                  | *（可选）* pollingInterval 的单位是秒。这是 KEDA 检查触发器的队列长度或流滞后的时间间隔。默认是 `30` 秒。 |
| **cooldownPeriod** *int32*                                   | *（可选）* cooldownPeriod 也是以秒为单位，它是在最后一个触发器激活后等待的时间段，然后再缩减到 0。 默认是 `300` 秒。 |
| **minReplicaCount** *int32*                                  | *（可选）* KEDA 在收缩资源的最小副本数。默认情况下为 `0`。   |
| **maxReplicaCount** *int32*                                  | *（可选）* KEDA 会将该值传递给为资源创建的 HPA 定义。        |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | *（可选）* 此属性指定在删除 “ScaledObject” 后，目标资源（“Deployments”、“StatefulSet”...）是否应被缩减到原始副本数量。默认行为是保持复制数量与删除 “ScaledObject” 时的数量相同。参考 [kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig) |
| **triggers** *\[][kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers)* | 触发工作负载动态伸缩的事件源，参考 [kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers) |

### KedaScaledJob

*从属 [Keda](#keda)*

> 你可以参考 [Scaling Jobs](https://keda.sh/docs/2.4/concepts/scaling-jobs/) 获得更多信息

| 字段                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **restartPolicy** *[v1.RestartPolicy](https://pkg.go.dev/k8s.io/api/core/v1#RestartPolicy)* | 在 pod 内所有容器的重启策略。可选择 `OnFailure`、`Never`。默认为 `Never`。 |
| **pollingInterval** *int32*                                  | *（可选）* pollingInterval 的单位是秒。这是 KEDA 检查触发器的队列长度或流滞后的时间间隔。默认是 `30` 秒。 |
| **successfulJobsHistoryLimit** *int32*                       | *（可选）* 应该保留多少个已完成的 Job。默认为 `100`。        |
| **failedJobsHistoryLimit** *int32*                           | *（可选）* 应该保留多少个失败的 Job。默认为 `100`。          |
| **maxReplicaCount** *int32*                                  | *（可选）* 在一个轮询周期内可以创建的最大 pod 的数量。       |
| **scalingStrategy** *[kedav1alpha1.ScalingStrategy](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScalingStrategy)* | *（可选）* 选择一个缩放策略。可能的值是 `default`、`custom`、`accurate`。默认值是 `default`。参考 [kedav1alpha1.ScalingStrategy](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScalingStrategy) |
| **triggers** *\[][kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers)* | 触发工作负载动态伸缩的事件源，参考[kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers) |

