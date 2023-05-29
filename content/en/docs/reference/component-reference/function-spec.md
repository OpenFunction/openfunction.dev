---
title: "Function Specifications"
linkTitle: "Function Specifications"
weight: 6110
description: >	
    Learn about Function Specifications.
---

This document describes the specifications of the Function CRD.

## Function.spec

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **image** | string | Image upload path, e.g. demorepo/demofunction:v1 | true |
| [build](#functionspecbuild) | object | Builder specification for the function | false |
| **imageCredentials** | object | Credentials for accessing the image repository, refer to v1.LocalObjectReference | false |
| [serving](#functionspecserving) | object | Serving specification for the function | false |
| **version** | string | Function version, e.g. v1.0.0 | false |
| **workloadRuntime** | string | WorkloadRuntime for Function. Know values: OCIContainer and WasmEdge.Default: OCIContainer | false |

### Function.spec.build
<sup><sup>[↩ Parent](#functionspec)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [srcRepo](#functionspecbuildsrcrepo) | object | The configuration of the source code repository | true |
| **builder** | string | Name of the Builder | false |
| **builderCredentials** | object | Credentials for accessing the image repository, refer to [v1.LocalObjectReference](https://pkg.go.dev/k8s.io/api/core/v1#LocalObjectReference) | false |
| **builderMaxAge** | string | The maximum time of finished builders to retain. | false |
| **dockerfile** | string | Path to the Dockerfile instructing Shipwright when using the Dockerfile to build images | false |
| **env** | map[string]string | Environment variables passed to the buildpacks builder | false |
| **failedBuildsHistoryLimit** | integer | The number of failed builders to retain. Default is 1. | false |
| [shipwright](#functionspecbuildshipwright) | object | Specification of the Shipwright engine | false |
| **successfulBuildsHistoryLimit** | integer | The number of successful finished builders to retain. Default is 0. | false |
| **timeout** | string | The maximum time for the builder to build the image | false |

### Function.spec.build.srcRepo
<sup><sup>[↩ Parent](#functionspecbuild)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [bundleContainer](#functionspecbuildsrcrepobundlecontainer) | object | BundleContainer describes the source code bundle container to pull | false |
| **credentials** | object | Repository access credentials, refer to v1.LocalObjectReference | false |
| **revision** | string | Referencable instances in the repository, such as commit ID and branch name. | false |
| **sourceSubPath** | string | The directory of the function in the repository, e.g. functions/function-a/ | false |
| **url** | string | Source code repository address | false |

### Function.spec.build.srcRepo.bundleContainer
<sup><sup>[↩ Parent](#functionspecbuildsrcrepo)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **image** | string | The bundleContainer image name | true |

### Function.spec.build.shipwright
<sup><sup>[↩ Parent](#functionspecbuild)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **params** | []object | Parameters for the build strategy | false |
| [strategy](../../../concepts/build_strategy) | object | Strategy references the BuildStrategy to use to build the image | false |
| **timeout** | string | The maximum amount of time the shipwright Build should take to execute | false |

### Function.spec.serving
<sup><sup>[↩ Parent](#functionspec)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **annotations** | map[string]string | Annotations that will be added to the workload | false |
| **bindings** | map[string]object | [Dapr bindings](https://docs.dapr.io/developing-applications/building-blocks/bindings/) that the function needs to create and use. | false |
| [hooks](#functionspecservinghooks) | object | Hooks that will be executed before or after the function execution | false |
| **labels** | map[string]string | Labels that will be added to the workload | false |
| [outputs](#functionspecservingoutputsindex) | []object | The outputs which the function will send data to | false |
| **params** | map[string]string | Parameters required by the function, will be passed to the function as environment variables | false |
| **pubsub** | map[string]object | [Dapr pubsub](https://docs.dapr.io/developing-applications/building-blocks/pubsub/) that the function needs to create and use | false |
| [scaleOptions](#functionspecservingscaleoptions) | object | Configuration of auto scaling. | false |
| [states](#functionspecservingstateskey) | map[string]object | Dapr state store that the function needs to create and use | false |
| **template** | object | Template is a pod template which allows modifying operator generated pod template. | false |
| **timeout** | string | Timeout defines the maximum amount of time the Serving should take to execute before the Serving is running | false |
| [tracing](#functionspecservingtracing) | object | Configuration of tracing | false |
| **triggers** | object | Triggers used to trigger the function. Refer to [Function Trigger](../../../concepts/function_trigger). | true |
| **workloadType** | string | The type of workload used to run the function, known values are: Deployment, StatefulSet and Job | false |

### Function.spec.serving.hooks
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required</th>
        </tr>
    </thead>
    <tbody><tr>
        <td><b>policy</b></td>
        <td>string</td>
        <td>
          There are two kind of hooks, global hooks and private hooks, the global hooks define in the config file of OpenFunction Controller, 
          the private hooks define in the Function. Policy is the relationship between the global hooks and the private hooks of the function. Known values are:
          <br/>
            <i><b>&nbsp;&nbsp;Append</b></i>: All hooks will be executed, the private pre hooks will execute after the global pre hooks , and the private post hooks will execute before the global post hooks. this is the default policy.<br/>
            <i><b>&nbsp;&nbsp;Override</b></i>: Only execute the private hooks.<br/>
        </td>
        <td>false</td>
      </tr><tr>
        <td><b>post</b></td>
        <td>[]string</td>
        <td>
          The hooks will be executed after the function execution
        </td>
        <td>false</td>
      </tr><tr>
        <td><b>pre</b></td>
        <td>[]string</td>
        <td>
          The hooks will be executed before the function execution
        </td>
        <td>false</td>
      </tr></tbody>
</table>

### Function.spec.serving.outputs[index]
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [dapr](#functionspecservingoutputsindexdapr) | object | Dapr output, refer to a exist component or a component defined in bindings or pubsub  | false |

### Function.spec.serving.outputs[index].dapr
<sup><sup>[↩ Parent](#functionspecservingoutputsindex)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **name** | string | The name of the dapr component | true |
| **metadata** | map[string]string | Metadata passed to Dapr | false |
| **operation** | string | Operation field tells the Dapr component which operation it should perform, refer to [Dapr docs](https://docs.dapr.io/reference/components-reference/supported-bindings/kafka/#binding-support) | false |
| **topic** | string | When the type is pubsub, you need to set the topic | false |
| **type** | string | Type of Dapr component, such as: bindings.kafka, pubsub.rocketmq | false |

### Function.spec.serving.scaleOptions
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [keda](#functionspecservingscaleoptionskeda) | object | Configuration about keda autoscaling | false |
| **knative** | map[string]string | Knative autiscaling annotations. Refer to [Knative autoscaling](https://knative.dev/docs/serving/autoscaling/). | false |
| **maxReplicas** | integer | Minimum number of replicas which will scale the resource down to. By default, it scales to 0. | false |
| **minReplicas** | integer | Maximum number of replicas which will scale the resource up to. | false |

### Function.spec.serving.scaleOptions.keda
<sup><sup>[↩ Parent](#functionspecservingscaleoptions)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [scaledJob](#functionspecservingscaleoptionskedascaledjob) | object | Scale options for job | false |
| [scaledObject](#functionspecservingscaleoptionskedascaledobject) | object | Scale options for deployment and statefulset | false |
| **triggers** | []object | Event sources that trigger dynamic scaling of workloads. Refer to [kedav1alpha1.ScaleTriggers](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#ScaleTriggers). | false |

### Function.spec.serving.scaleOptions.keda.scaledJob
<sup><sup>[↩ Parent](#functionspecservingscaleoptionskeda)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **failedJobsHistoryLimit** | integer | How many failed jobs should be kept. It defaults to 100. | false |
| **pollingInterval** | integer | The pollingInterval is in seconds. This is the interval in which KEDA checks the triggers for the queue length or the stream lag. It defaults to 30 seconds. | false |
| **restartPolicy** | string | Restart policy for all containers within the pod. Value options are OnFailure or Never. It defaults to Never. | false |
| **scalingStrategy** | object | Select a scaling strategy. Value options are default, custom, or accurate. The default value is default. Refer to kedav1alpha1.ScalingStrategy | false |
| **successfulJobsHistoryLimit** | integer | How many completed jobs should be kept. It defaults to 100. | false |

### Function.spec.serving.scaleOptions.keda.scaledObject
<sup><sup>[↩ Parent](#functionspecservingscaleoptionskeda)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **advanced** | object | This property specifies whether the target resource (for example, Deployment and StatefulSet) should be scaled back to original replicas count after the ScaledObject is deleted. Default behavior is to keep the replica count at the same number as it is in the moment of ScaledObject deletion. Refer to [kedav1alpha1.AdvancedConfig](https://pkg.go.dev/github.com/kedacore/keda/v2/api/v1alpha1#AdvancedConfig). | false |
| **cooldownPeriod** | integer | The cooldownPeriod is in seconds, and it is the period of time to wait after the last trigger activated before scaling back down to 0. It defaults to 300 seconds. | false |
| **pollingInterval** | integer | The pollingInterval is in seconds. This is the interval in which KEDA checks the triggers for the queue length or the stream lag. It defaults to 30 seconds. | false |

### Function.spec.serving.states[key]
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **spec** | object | Dapr state stroe component spec. Refer to [Dapr docs](https://docs.dapr.io/developing-applications/building-blocks/state-management/). | false |

### Function.spec.serving.tracing
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **baggage** | map[string]string | Baggage is contextual information that passed between spans. It is a key-value store that resides alongside span context in a trace, making values available to any span created within that trace. | true |
| **enabled** | boolean | Wether to enable tracing | true |
| [provider](#functionspecservingtracingprovider) | object | The tracing implementation used to create and send span | true |
| **tags** | map[string]string | The tag that needs to be added to the spans | false |

### Function.spec.serving.tracing.provider
<sup><sup>[↩ Parent](#functionspecservingtracing)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **name** | string | Tracing provider name, known values are skywalking and opentelemetry | true |
| [exporter](#functionspecservingtracingproviderexporter) | object | Service to collect span for opentelemetry | false |
| **oapServer** | string | The skywalking server url | false |

### Function.spec.serving.tracing.provider.exporter
<sup><sup>[↩ Parent](#functionspecservingtracingprovider)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **endpoint** | string | The exporter url | true |
| **name** | string | The exporter name, known values are otlp, jaeger, and zipkin | true |
| **compression** | string | The compression type to use on OTLP trace requests. Options include gzip. By default no compression will be used. | false |
| **headers** | string | Key-value pairs separated by commas to pass as request headers on OTLP trace requests. | false |
| **protocol** | string | The transport protocol to use on OTLP trace requests. Options include grpc and http/protobuf. Default is grpc. | false |
| **timeout** | string | The maximum waiting time, in milliseconds, allowed to send each OTLP trace batch. Default is 10000. | false |

### Function.spec.serving.triggers
<sup><sup>[↩ Parent](#functionspecserving)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [dapr](#functionspecservingtriggersdaprindex) | []object | List of dapr triggers, refer to dapr bindings or pusub components | false |
| [http](#functionspecservingtriggershttp) | object | The http trigger | false |
| [inputs](#functionspecservingtriggersinputsindex) | []object | A list of components that the function can get data from | false |

### Function.spec.serving.triggers.dapr[index]
<sup><sup>[↩ Parent](#functionspecservingtriggers)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **name** | string | The dapr component name | true |
| **topic** | string | When the component type is pubsub, you need to set the topic | false |
| **type** | string | Type of Dapr component, such as: bindings.kafka, pubsub.rocketmq | false |

### Function.spec.serving.triggers.http
<sup><sup>[↩ Parent](#functionspecservingtriggers)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **port** | integer | The port the function is listening on, e.g. 8080 | false |
| [route](#functionspecservingtriggershttproute) | object | Route defines how traffic from the Gateway listener is routed to a function. | false |

### Function.spec.serving.triggers.http.route
<sup><sup>[↩ Parent](#functionspecservingtriggershttp)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [gatewayRef](#functionspecservingtriggershttproutegatewayref) | object | GatewayRef references the Gateway resources that a Route wants | false |
| **hostnames** | []string | Hostnames defines a set of hostname that should match against the HTTP Host header to select a HTTPRoute to process the request. | false |
| **rules** | []object | Rules are a list of HTTP matchers, filters and actions. Refer to [HTTPRouteRule](https://gateway-api.sigs.k8s.io/v1alpha2/references/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule). | false |

### Function.spec.serving.triggers.http.route.gatewayRef
<sup><sup>[↩ Parent](#functionspecservingtriggershttproute)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **name** | string | The name of the gateway | true |
| **namespace** | string | The namespace of the gateway | true |

### Function.spec.serving.triggers.inputs[index]
<sup><sup>[↩ Parent](#functionspecservingtriggers)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| [dapr](#functionspecservingtriggersinputsindexdapr) | object | A dapr component that function can get data from. Now just support dapr state store | true |

### Function.spec.serving.triggers.inputs[index].dapr
<sup><sup>[↩ Parent](#functionspecservingtriggersinputsindex)</sup></sup>

| Name | Type | Description | Required |
| --- | --- | --- | --- |
| **name** | string | The dapr component name, maybe a exist component or a component defined in state | true |
| **type** | string | The dapr component type, such as state.redis | false |

