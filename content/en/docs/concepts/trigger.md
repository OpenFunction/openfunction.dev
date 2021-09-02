---
title: "Trigger"
linkTitle: "Trigger"
weight: 35
description: >
    Concept of OpenFunction, a resource of the events framework, Trigger
---

**Trigger** is an abstraction of the purpose of an event, i.e. what needs to be done when a message is received.

**Trigger** contains the user's description of the purpose of the event, which guides the Trigger on which event sources it should fetch events from and subsequently decide whether to trigger the target function based on the given conditions.

### Reference

Trigger is a [CustomResourceDefinitions(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) . You can refer to [Trigger CRD Spec]({{< ref "../reference/trigger-spec" >}}) for more information. 
