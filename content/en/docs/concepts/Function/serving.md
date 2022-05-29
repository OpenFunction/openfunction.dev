---
title: "Serving"
linkTitle: "Serving"
weight: 3300
description: >	
    Get familiar with how functions are running in OpenFunction.
---

## Serving

Once a function is created with `Serving` spec, a `Serving` custom resource will be created to control a function's serving phase. Currently OpenFunction Serving supports two runtimes: the Knative sync runtime and the OpenFunction async runtime.

### The Knative sync runtime

For sync functions, OpenFunction currently supports using Knative Serving as runtime.
And we're planning to add another sync function runtime powered by the KEDA http-addon.

### The OpenFunction async runtime

The OpenFunction asyn runtime is an event-driven runtime which is implemented based on [KEDA](https://keda.sh/) and [Dapr](https://dapr.io/). Async functions can be triggered by various event types like message queue, cronjob, and MQTT etc.

<div align=center><img src=/images/docs/en/concepts/function/openfunction-serving.svg/></div>
