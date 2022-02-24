---
title: "Introduction"
linkTitle: "Introduction"
weight: 1000
description: >	
    Introduces the OpenFunction project.
---

This document introduces what is OpenFunction.

## Overview

OpenFunction is a cloud-native open source FaaS (Function as a Service) platform aiming to let you focus on your business logic without having to maintain the underlying runtime environment and infrastructure. You can generate event-driven and dynamically scaling Serverless workloads by simply submitting business-related source code in the form of functions.

![](/images/docs/en/introduction/what-is-openfunction/function-lifecycle.svg)

## Architecture and Design

![](/images/docs/en/introduction/what-is-openfunction/openfunction-0.5-architecture.png)

OpenFunction manages resources in the form of Custom Resource Definitions (CRDs) throughout the lifecycle of a function. 

## Core Features

- Converts business-related function source code to application source code.
- Generates OCI-compliant container images from the converted application source code.
- Deploys generated container images to any runtime with dynamic scaling capabilities.
- Provides events framework to make functions event-driven.
- Provides function version management and ingress management.

OpenFunction has two main capabilities:

- Serverless framework with CRDs: [Function](../../concepts/function), [Builder](../../concepts/builder), and [Serving](../../concepts/serving).
- Events framework with CRDs: [EventSource](../../concepts/events#eventsource), [EventBus (ClusterEventBus)](../../concepts/events#eventbus-clustereventbus), and [Trigger](../../concepts/events#trigger).

## Use Cases

To name a few use cases:

- Web applications
- Event-driven data processing
- Stream processing
- Serverless workflows

## License

OpenFunction is licensed under the Apache License, Version 2.0. For more information, see [LICENSE](https://github.com/OpenFunction/OpenFunction/blob/main/LICENSE).

