---
title: "Nats Streaming"
linkTitle: "Nats Streaming"
weight: 15
description: >
    Event bus specification of Nats Streaming
---

### NatsStreamingSpec

*Belong to [EventBusSpec]({{<ref "_index.md#eventbusspec" >}})*

> The EventBus(ClusterEventBus) will provide the configuration to the EventSource and Trigger references in order to generate the corresponding [Dapr Pub/Sub Components](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#component-format) to get messages from the event bus, and in principle we try to maintain consistency in the relevant parameters. You can get more information by visiting the [NATS Streaming pubsub spec](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#spec-metadata-fields).

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **natsURL** *string*                                         | NATS server address, e.g. `nats://localhost:4222`            |
| **natsStreamingClusterID** *string*                          | NATS cluster ID, e.g. `stan`                                 |
| **subscriptionType** *string*                                | Subscriber type, optional: `topic`, `queue`                  |
| **ackWaitTime** *string*                                     | *(Optional)* Refer to [Acknowledgements](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) , e.g. `300ms` |
| **maxInFlight** *int64*                                      | *(Optional)* Refer to [Max In Flight](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) , e.g. `25` |
| **durableSubscriptionName** *string*                         | *(Optional)* The name of the [persistent subscriber](https://docs.nats.io/developing-with-nats-streaming/durables). E.g. `my-durable` |
| **deliverNew** *bool*                                        | *(Optional)* Subscriber options (only one can be used). Whether to send only new messages. Optional: `true`, `false` |
| **startAtSequence** *int64*                                  | *(Optional)* Subscriber options (only one can be used). Set the starting sequence position and status. E.g. `100000` |
| **startWithLastReceived** *bool*                             | *(Optional)* Subscriber options (only one can be used). Whether to set the start position to the latest news place. Optional: `true`, `false` |
| **deliverAll** *bool*                                        | *(Optional)* Subscriber options (only one can be used). Whether to send all available messages. Optional: `true`, `false` |
| **startAtTimeDelta** *string*                                | *(Optional)* Subscriber options (only one can be used). Use the difference form to set the desired start time position and state, e.g. `10m`, `23s` |
| **startAtTime** *string*                                     | *(Optional)* Subscriber options (only one can be used). Set the desired start time position and status using the marker value form. E.g. `Feb 3, 2013 at 7:54pm (PST)` |
| **startAtTimeFormat** *string*                               | *(Optional)* Must be used with startAtTime. Sets the format of the time. E.g. `Jan 2, 2006 at 3:04pm (MST)` |
| **scaleOption** [NatsStreamingScaleOption](#natsstreamingscaleoption) | *(Optional)* Nats streaming's scale configuration            |

### NatsStreamingScaleOption

*Belong to [NatsStreamingSpec](#natsstreamingspec)*

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| *[GenericScaleOption]({{< ref "../eventbus-spec#genericscaleoption" >}})* | Generic scale configuration                                  |
| **natsServerMonitoringEndpoint** *string*                    | Nats streaming's monitoring endpoint                         |
| **queueGroup** *string*                                      | Nats streaming's queue group name                            |
| **durableName** *string*                                     | Nats streaming's durable name                                |
| **subject** *string*                                         | Subject under monitoring, e.g. `topicA`, `myTopic`           |
| **lagThreshold** *string*                                    | Threshold for triggering scaling, in this case is the Nats Streaming's lag |

