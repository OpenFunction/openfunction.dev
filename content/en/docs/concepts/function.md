---
title: "Function"
linkTitle: "Function"
weight: 3100
description: >	
    Learn about the concept of Function in OpenFunction.
---

## Function

`Function` is the control plane of `Build` and `Serving` and it's also the interface for users to use OpenFunction. Users needn't to create the `Build` or `Serving` separately because `Function` is the only place to define a function's `Build` and `Serving`.

Once a function is created, it will controll the lifecycle of `Build` and `Serving` without user intervention:

- If `Build` is defined in a function, a builder custom resource will be created to build function's container image once a function is deployed.

- If `Serving` is defined in a function, a serving custom resource will be created to control a function's serving and autoscalling.

- `Build` and `Serving` can be defined together which means the function image will be built first and then it will be used in serving.

- `Build` can be defined without `Serving`, the function is used to build image only in this case.

- `Serving` can be defined without `Build`, the function will use a previously built function image for serving.

<div align=center><img width="70%" height="70%" src=/images/docs/en/introduction/what-is-openfunction/openfunction-0.5-architecture.svg/></div>

## Reference

For more information, see [Function Specifications](../../../reference/component-reference/function-spec).