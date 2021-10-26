---
title: "Function Spec"
linkTitle: "Function Spec"
weight: 5
description: >
    Function CRD Specification
---

## Function

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **apiVersion** *string*                                      | core.openfunction.io/v1alpha2                                |
| **kind** *string*                                            | Function                                                     |
| **metadata** *[v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)* | *(Optional)* Refer to [v1.ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta) |
| **spec** *[FunctionSpec](#functionspec)*                     | Refer to [FunctionSpec](#functionspec)                       |
| **status** *FunctionStatus*                                  | Refer to FunctionStatus                                      |

### FunctionSpec

*Belong to [Function](#function)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **version** *string*                                         | *(Optional)* Function version, e.g. `v1.0.0`                 |
| **image** *string*                                           | Upload path for image, e.g. `demorepo/demofunction:v1`       |
| **imageCredentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *(Optional)* Credentials for accessing the image repository, refer to [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |
| **port** *int32*                                             | *(Optional)* The port the function is listening on, e.g. `8080` |
| **build** *[BuildImpl](#buildimpl)*                          | *(Optional)* Builder specification for the function, see [BuildImpl](#buildimpl) |
| **serving** *[ServingImpl](#servingimpl)*                    | *(Optional)* Serving specification for the function, see [ServingImpl](#servingimpl) |

### BuildImpl

*Belong to [FunctionSpec](#functionspec)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **builder** *string*                                         | Name of the Builder                                          |
| **builderCredentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *(Optional)* Credentials for accessing the image repository, refer to [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |
| **shipwright** *[ShipwrightEngine](#shipwrightengine)*       | *(Optional)* Specification of the Shipwright engine, refer to [ShipwrightEngine](#shipwrightengine) |
| **params** *map[string]string*                               | *(Optional)* Parameters passed to Shipwright                 |
| **env** map[string]string                                    | *(Optional)* Parameters passed to the buildpacks builder     |
| **srcRepo** *[GitRepo](#gitrepo)*                            | The configuration of the source code repository, refer to [GitRepo](#gitrepo) |
| **dockerfile** *string*                                      | *(Optional)* Path to the Dockerfile file to guide Shipwright in using Dockerfile to build images |

### ShipwrightEngine

*Belong to [BuildImpl](#buildimpl)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **strategy** *[Strategy](#strategy)*                         | *(Optional)* Index of image build strategiescy, refer to [Strategy](#strategy) |
| **timeout** *[v1.Duration](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#Duration)* | *(Optional)* Build timeout, refer to [v1.Duration](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#Duration) |

### Strategy

*Belong to [ShipwrightEngine](#shipwrightengine)*

| Field           | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| **name** string | Name of the strategy                                         |
| **kind** string | *(Optional)* Kind of build strategy, default is "BuildStrategy", optional "ClusterBuildStrategy" |

### GitRepo

*Belong to [BuildImpl](#buildimpl)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **url** *string*                                             | Source code repository address                               |
| **revision** *string*                                        | *(Optional)* Referenceable instances in the repository, such as commit id, branch name, etc. |
| **sourceSubPath** *string*                                   | *(Optional)* The directory of the function in the repository, e.g. `functions/function-a/` |
| **credentials** *[v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference)* | *(Optional)* Repository access credentials, refer to [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) |

### ServingImpl

*Belong to [FunctionSpec](#functionspec)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **runtime** *string*                                         | Type of load runtime, optional: `Knative`, `OpenFuncAsync`   |
| **params** *map[string]string*                               | *(Optional)* Parameters passed to the workloads              |
| **openFuncAsync** *[OpenFuncAsyncRuntime](#openfuncasyncruntime)* | *(Optional)* Used to define the configuration of OpenFuncAsync when the runtime is OpenFuncAsync, see [OpenFuncAsyncRuntime](#openfuncasyncruntime) |
| **template** *[v1.PodSpec](https://pkg.go.dev/k8s.io/api/core/v1#PodSpec)* | *(Optional)* Template for the definition of Pods in the workloads, refer to [v1.PodSpec](https://pkg.go.dev/k8s.io/api/core/v1#PodSpec) |

### OpenFuncAsyncRuntime

*Belong to [ServingImpl](#servingimpl)*

| Field                    | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **dapr** *[Dapr](#dapr)* | *(Optional)* Definition of Dapr components, see [Dapr](#dapr) |
| **keda** *[Keda](#keda)* | *(Optional)* Definition of Keda, see [Keda](#keda)           |

### Dapr

*Belong to [OpenFuncAsyncRuntime](#openfuncasyncruntime)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **annotations** *map[string]string*                          | *(Optional)* Annotations for Dapr components, see [Dapr docs](https://docs.dapr.io/reference/arguments-annotations-overview/) |
| **components** *map\[string][componentsv1alpha1.ComponentSpec](https://docs.dapr.io/reference/components-reference/)* | *(Optional)* Dapr Components Spec map, key is component's name, value is [componentsv1alpha1.ComponentSpec](https://docs.dapr.io/reference/components-reference/) |
| **inputs** *\[][DaprIO](#daprio)*                            | *(Optional)* The definition of the inputs of the function, see [DaprIO](#daprio) |
| **outputs** *\[][DaprIO](#daprio)*                           | *(Optional)* The definition of the outputs of the function, see [DaprIO](#daprio) |

### DaprIO

*Belong to [Dapr](#dapr)*

| Field                          | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| **name** *string*              | Name of the input and output of the function. Consistent with the name of [DaprComponent](#daprcomponent) means associated |
| **component** *string*         | Indicates the name of components                             |
| **type** *string*              | Type of Dapr component, optional: `bindings`, `pubsub`       |
| **topic** *string*             | *(Optional)* When the **type** is `pubsub`, you need to set the topic |
| **operation** *string*         | *(Optional)* Operation field tells the Dapr component which operation it should perform, refer to [Dapr docs](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#binding-support) |
| **params** *map[string]string* | *(Optional)* Parameters passed to Dapr                       |

### Keda

*Belong to [OpenFuncAsyncRuntime](#openfuncasyncruntime)*

| Field                                                    | Description                                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| **scaledObject** *[KedaScaledObject](#kedascaledobject)* | Definition of KEDA scalable objects(Deployments), refer to [KedaScaledObject](#kedascaledobject) |
| **scaledJob** *[KedaScaledJob](#kedascaledjob)*          | Definition of KEDA scalable jobs, refer to [KedaScaledJob](#kedascaledjob) |

### KedaScaledObject

*Belong to [Keda](#keda)*

> You can refer to [Scaling Deployments, StatefulSets & Custom Resources](https://keda.sh/docs/2.4/concepts/scaling-deployments/) for more information

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **workloadType** *string*                                    | How to run the function, known values are `Deployment` or `StatefulSet`, default is `Deployment`. |
| **pollingInterval** *int32*                                  | *(Optional)* The pollingInterval is in seconds. This is the interval in which KEDA checks the triggers for the queue length or the stream lag. Default is `30` seconds. |
| **cooldownPeriod** *int32*                                   | *(Optional)* The cooldownPeriod is also in seconds, and it is the period of time to wait after the last trigger activated before scaling back down to 0. Default is `300` seconds. |
| **minReplicaCount** *int32*                                  | *(Optional)* Minimum number of replicas KEDA will scale the resource down to. By default it’s scale to `0`. |
| **maxReplicaCount** *int32*                                  | *(Optional)* This setting is passed to the HPA definition that KEDA will create for a given resource. |
| **advanced** *[kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig)* | *(Optional)* This property specifies whether the target resource (`Deployment`, `StatefulSet`,…) should be scaled back to original replicas count, after the `ScaledObject` is deleted. Default behavior is to keep the replica count at the same number as it is in the moment of `ScaledObject's` deletion. Refer to [kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig) |
| **triggers** *\[][kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers)* | Event sources that trigger dynamic scaling of workloads, refer to [kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers) |

### KedaScaledJob

*Belong to [Keda](#keda)*

> You can refer to [Scaling Jobs](https://keda.sh/docs/2.4/concepts/scaling-jobs/) for more information

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **restartPolicy** *[v1.RestartPolicy](https://pkg.go.dev/k8s.io/api/core/v1#RestartPolicy)* | Restart policy for all containers within the pod. Optional `OnFailure`, `Never`. Default is `Never`. |
| **pollingInterval** *int32*                                  | *(Optional)* The pollingInterval is in seconds. This is the interval in which KEDA checks the triggers for the queue length or the stream lag. Default is `30` seconds. |
| **successfulJobsHistoryLimit** *int32*                       | *(Optional)* How many completed jobs should be kept.. Default is `100` . |
| **failedJobsHistoryLimit** *int32*                           | *(Optional)* How many failed jobs should be kept. Default is `100` . |
| **maxReplicaCount** *int32*                                  | *(Optional)* The max number of pods that is created within a single polling period. |
| **scalingStrategy** *[kedav1alpha1.ScalingStrategy](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScalingStrategy)* | *(Optional)* Select a Scaling Strategy. Possible values are `default`, `custom`, or `accurate`. The default value is `default`.. Refer to [kedav1alpha1.ScalingStrategy](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScalingStrategy) |
| **triggers** *\[][kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers)* | Event sources that trigger dynamic scaling of workloads, refer to [kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers) |

