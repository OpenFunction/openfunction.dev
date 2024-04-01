---
title: "Introduction"
linkTitle: "Introduction"
weight: 1000
description: 
---
## Overview

OpenFunction is a user-friendly, cloud-native FaaS (Function as a Service) platform. Its aim is to let you concentrate on your business logic without the hassle of managing the runtime environment and infrastructure. With OpenFunction, you effortless create event-driven and dynamically scaling Serverless workloads by submitting business-related source code in the form of functions.

<div align=center><img width="70%" height="70%" src=/images/docs/en/introduction/what-is-openfunction/function-lifecycle.svg/></div>


## Architecture and Design

<div align=center><img width="80%" height="80%" src=/images/docs/en/introduction/what-is-openfunction/openfunction-architecture.svg/></div>

## Core Features

- Cloud-agnostic and decoupled from cloud providers' BaaS
- Pluggable architecture that allows multiple function runtimes
- Supports both sync and async functions
- Unique async functions support that can consume events directly from event sources
- Supports generating OCI-Compliant container images directly from function source code.
- Flexible autoscaling between 0 and N
- Advanced async function autoscaling based on event sources' specific metrics
- Simplified BaaS integration for both sync and async functions by introducing [Dapr](https://dapr.io/)
- Advanced function ingress & traffic management powered by [K8s Gateway API](https://gateway-api.sigs.k8s.io/) (In Progress)
- Flexible and easy-to-use events management framework

## License

OpenFunction is licensed under the Apache License, Version 2.0. For more information, see [LICENSE](https://github.com/OpenFunction/OpenFunction/blob/main/LICENSE).

