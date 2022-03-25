---
title: "Announcing OpenFunction 0.6.0: FaaS observability, HTTP trigger, and more"
linkTitle: "Release v0.4.0"
date: 2022-03-25
weight: 100

---

OpenFunction is a cloud-native open source FaaS platform aiming to let you focus on your business logic. The OpenFunction community has been hard at work over the past several months preparing for the v0.6.0 release. Today, we proudly announce v0.6.0 is officially available. Thanks to the community for helping drive the new features, enhancements, and bug fixes.

OpenFunction 0.6.0 brings notable features including function plugin, distributed tracing for functions, control autoscaling behavior, HTTP trigger to async function, etc. Meanwhile, OpenFuncAsync runtime definition has also been refactored. The core API has been upgraded from v1beta1 to v1alpha1. 

### Distributed tracing for serverless functions

When trying to understand and diagnose the distributed systems and microservices, one of the most effective methods is the stack trace. Distributed tracing provides a holistic view of the way that messages flow and distributed transaction monitoring for Serverless functions. The OpenFunction team collaborates with the Apache SkyWalking community to add FaaS observability which allows you to visualize function dependencies and track function invocations on SkyWalking UI. 

![openfunction-tracing-skywalking](images/docs/en/blogs/openfunction-tracing-skywalking.jpg)

Going forward, OpenFunction will add more capabilities for serverless functions in logging, metrics, and tracing. You will be able to use Apache SkyWalking and OpenFunction to set up a full-stack APM for your serverless workloads out-of-the-box. Moreover, OpenFunction will support OpenTelemetry that allows you to leverage Jaeger or Zipkin as an alternative.

### Support Dapr pub/sub and bindings

Dapr bindings allows you to trigger your applications or services with events coming in from external systems, or interface with external systems. OpenFunction 0.6.0 adds Dapr output bindings to its synchronous functions which enables HTTP triggers for asynchronous functions. For example, synchronous functions backed by Knative runtime can now interact with middleware via Dapr components just act like the Async functions. See this [guide](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding) for the quickstart sample.

​​OpenFuncAsync functions introduces [Dapr pub/sub](https://docs.dapr.io/developing-applications/building-blocks/pubsub/pubsub-overview/) to provide a platform-agnostic API to send and receive messages. A typical use case is that you can leverage synchronous functions to receive an event in plain JSON or Cloud Events format, then send the received event to a Dapr output binding or pub/sub component, most likely a message queue (e.g. Kafka, Redis Stream, RabbitMQ). Finally, the OpenFuncAsync function could be triggered from the message queue. See this [guide](https://github.com/OpenFunction/samples/tree/main/functions/async/pubsub) for the quickstart sample.

![http-trigger-openfunction](images/docs/en/blogs/http-trigger-openfunction.jpg)

### Control autoscaling behavior for functions

OpenFunction 0.6.0 integrates KEDA [ScaledObject](https://keda.sh/docs/2.5/concepts/scaling-deployments/#scaledobject-spec) spec which is used to define how KEDA should scale your application and what the triggers are. You just need to define the lower and upper bound in the function CRD without changing your code. 

Meanwhile, it also provides the ability to control the concurrency, the number of simultaneous requests, which inherits the definition from [Dapr](https://docs.dapr.io/operations/configuration/control-concurrency/) and [Knative](https://knative.dev/docs/serving/autoscaling/concurrency/). A typical use case in distributed computing is to only allow for a given number of requests to execute concurrently. You are able to control how many requests and events will invoke your application simultaneously.

### Learn by doing

Benjamin Huo, the founder of OpenFunction, has presented two typical use cases of OpenFunction 0.6.0 in Dapr community meeting:

1. [HTTP trigger for asynchronous functions with OpenFunction and Kafka](https://github.com/OpenFunction/samples/tree/main/functions/knative/with-output-binding)
2. [Elastic Kubernetes log alerts with OpenFunction and Kafka](https://github.com/OpenFunction/samples/tree/main/functions/knative/logs-handler-function)

Watch this video and follow with the hands-on guides to practice.

You can learn more about OpenFunction 0.6.0 from [release notes](https://github.com/OpenFunction/OpenFunction/releases/tag/v0.6.0). Get started with OpenFunction by following the [Quick Start](https://github.com/OpenFunction/OpenFunction#-quickstart) and [samples](https://github.com/OpenFunction/samples).