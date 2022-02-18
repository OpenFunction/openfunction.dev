---
title: "NATS Streaming"
linkTitle: "NATS Streaming"
weight: 6132
description: >
    Event bus specifications of NATS Streaming.
---

### NatsStreamingSpec

*Belong to [EventBusSpec](../eventbus-spec)*.

{{% alert title="Note" color="success" %}}

The EventBus (ClusterEventBus) provides the configuration to the EventSource and Trigger references in order to generate the corresponding [Dapr Pub/Sub Components](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#component-format) to get messages from the event bus, and in principle we try to maintain consistency in the relevant parameters. For more information, see the [NATS Streaming pubsub spec](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-nats-streaming/#spec-metadata-fields).

{{% /alert %}}

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **natsURL** *string*                                         | NATS server address, for example, `nats://localhost:4222`.   |
| **natsStreamingClusterID** *string*                          | NATS cluster ID, for example, `stan`.                        |
| **subscriptionType** *string*                                | Subscriber type, value options: `topic`, `queue`.            |
| **ackWaitTime** *string*                                     | *(Optional)* Refer to [Acknowledgements](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) , for example, `300ms`. |
| **maxInFlight** *int64*                                      | *(Optional)* Refer to [Max In Flight](https://docs.nats.io/developing-with-nats-streaming/acks#acknowledgements) , for example, `25`. |
| **durableSubscriptionName** *string*                         | *(Optional)* The name of the [persistent subscriber](https://docs.nats.io/developing-with-nats-streaming/durables). For example, `my-durable`. |
| **deliverNew** *bool*                                        | *(Optional)* Subscriber options (only one can be used). Whether to send only new messages. Value options: `true`, `false`. |
| **startAtSequence** *int64*                                  | *(Optional)* Subscriber options (only one can be used). Set the starting sequence position and status. For example, `100000`. |
| **startWithLastReceived** *bool*                             | *(Optional)* Subscriber options (only one can be used). Whether to set the start position to the latest news place. Value options: `true`, `false`. |
| **deliverAll** *bool*                                        | *(Optional)* Subscriber options (only one can be used). Whether to send all available messages. Value options: `true`, `false`. |
| **startAtTimeDelta** *string*                                | *(Optional)* Subscriber options (only one can be used). Use the difference form to set the desired start time position and state, for example, `10m`, `23s`. |
| **startAtTime** *string*                                     | *(Optional)* Subscriber options (only one can be used). Set the desired start time position and status using the marker value form. For example, `Feb 3, 2013 at 7:54pm (PST)`. |
| **startAtTimeFormat** *string*                               | *(Optional)* Must be used with startAtTime. Sets the format of the time. For example, `Jan 2, 2006 at 3:04pm (MST)`. |
| **scaleOption** [NatsStreamingScaleOption](#natsstreamingscaleoption) | *(Optional)* Nats streaming's scale configuration.           |

### NatsStreamingScaleOption

*Belong to [NatsStreamingSpec](#natsstreamingspec)*.

| Field                                                        | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| *[GenericScaleOption]({{< ref "../eventbus-spec#genericscaleoption" >}})* | Generic scale configuration.                                 |
| **natsServerMonitoringEndpoint** *string*                    | Nats streaming's monitoring endpoint.                        |
| **queueGroup** *string*                                      | Nats streaming's queue group name.                           |
| **durableName** *string*                                     | Nats streaming's durable name.                               |
| **subject** *string*                                         | Subject under monitoring, for example, `topicA`, `myTopic`.  |
| **lagThreshold** *string*                                    | Threshold for triggering scaling, in this case is the Nats Streaming's lag. |

