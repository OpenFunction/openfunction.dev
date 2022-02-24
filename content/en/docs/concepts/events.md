---
title: "Events"
linkTitle: "Events"
weight: 3400
description: >	
    Learn about the concept of Events in OpenFunction.
---

This document describes the concept of Events in OpenFunction.

## Overview

Events is the event management framework for OpenFunction. It provides the following core features:

- Support for triggering target functions by synchronous and asynchronous calls
- User-defined trigger judgment logic
- The components of OpenFunction Events can be driven by OpenFunction itself啥意思？

## Architecture

The following diagram illustrates the architecture of Events.

![of-events-architecture](/images/docs/en/concepts/events/of-events-architecture.svg)

## Concepts

### EventSource

EventSource defines the producer of an event, such as a Kafka service, an object storage service, and even a function. It contains descriptions of these event producers and information about where to send these events.

EventSource supports the following types of event source server:

- Kafka
- Cron (scheduler)
- Redis

### EventBus (ClusterEventBus)

EventBus is responsible for aggregating events and making them persistent. It contains descriptions of an event bus broker that usually is a message queue (such as NATS Streaming and Kafka), and provides these configurations for EventSource and Trigger.

EventBus handles event bus adaptation for namespace scope by default. For cluster scope, ClusterEventBus is available as an event bus adapter and takes effect when other components cannot find an EventBus under a namespace.

EventBus supports the following event bus broker:

- NATS Streaming

### Trigger

Trigger is an abstraction of the purpose of an event, such as what needs to be done when a message is received. It contains the purpose of an event defined by you, which tells the trigger which EventSource it should fetch the event from and subsequently determine whether to trigger the target function according to the given conditions.

## Reference

For more information, see [EventSource Specifications](../../reference/component-reference/eventsource-spec/eventsource-spec) and [EventBus Specifications](../../reference/component-reference/eventbus-spec/eventbus-spec).

