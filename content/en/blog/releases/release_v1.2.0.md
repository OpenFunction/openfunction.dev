---
title: "OpenFunction 1.2.0: integrating KEDA http-addon as a synchronous function runtime"
linkTitle: "Release v1.2.0"
date: 2023-11-01
weight: 92

---

[OpenFunction](https://github.com/OpenFunction/OpenFunction) is an open-source cloud-native FaaS (Function as a Service) platform designed to help developers focus on their business logic. We are thrilled to announce another important update for OpenFunction, the release of version v1.2.0!

In this update, we continue to strive to provide developers with more flexible and powerful tools, and have added some new features. This version integrates KEDA http-addon as a synchronous function runtime, supports adding environment variables when enabling SkyWalking tracing, and supports recording build time. Additionally, several components have been upgraded and multiple bugs have been fixed.

Here are the main highlights of this version update:

## integrating KEDA http-addon as a synchronous function runtime

KEDA http-addon is an additional component of KEDA that automatically scales HTTP servers based on changes in HTTP traffic, including scaling from zero to handle traffic and scaling down to zero when there is no traffic.

The working principle of KEDA http-addon is that it creates a component called Interceptor in the Kubernetes cluster to receive all HTTP requests and forward them to the target application. At the same time, it reports the length of the request queue to a component called External Scaler, which triggers KEDA's automatic scaling mechanism. This allows your HTTP application to dynamically adjust the number of replicas based on the actual traffic demand.

In OpenFunction version v1.2.0, we have integrated KEDA http-addon as an option for synchronous function runtime. This means that you can use OpenFunction to create and manage HTTP-based functions and leverage the capabilities of KEDA http-addon for efficient and flexible elastic scaling. To deploy and run your HTTP functions, you simply need to specify the value of `serving.triggers[*].http.engine`  as "keda" when creating the Function resource and configure the relevant parameters of `keda.httpScaledObject` in `serving.scaleOptions`.

## Support for recording events when the states of Function, Builder, and Serving change

Events are an important resource type in Kubernetes that can record important or interesting occurrences within a cluster. Events can help users and developers understand changes in the state of resources within the cluster and handle any abnormalities accordingly.

In OpenFunction version v1.2.0, we support recording events when the states of Function, Builder, and Serving change. This allows you to gain more information about what happens during the function building and running processes by reviewing the events. For example, you can see events such as the start, end, or failure of function building, as well as events related to the creation, update, or deletion of function runtimes.

## Other improvements and optimizations

In addition to the major changes mentioned above, this version also includes the following modifications and enhancements:

* Upgraded KEDA to v2.10.1 and HPA (Horizontal Pod Autoscaler) API version to v2, improving stability and compatibility.
* Added support for recording build time, allowing you to track the duration of function builds.
* Adjusted the CI (Continuous Integration) process and fixed some minor issues.
* Fixed a bug in the keda http-addon runtime that caused functions to not run properly.
* Upgraded several components in the charts, including keda, dapr, and contour, to ensure the use of the latest versions and features.
* These are the main functional changes in OpenFunction v1.2.0. We would like to express our sincere gratitude to all contributors for their participation and contributions.

To learn more about OpenFunction and this version update, please visit our official website and GitHub page.

- [Official Website](https://openfunction.dev/)：https://openfunction.dev/
- [Github](https://github.com/OpenFunction/OpenFunction/releases/tag/v1.2.0)：https://github.com/OpenFunction/OpenFunction/releases/tag/v1.2.0