---
title: "Cron"
linkTitle: "Cron"
weight: 6124
description: >
    Event source specifications of Cron.
---

### CronSpec

*Belong to [EventSourceSpec](../eventsource-spec)*.

{{% alert title="Note" color="success" %}}

The EventSource generates [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#component-format) for adapting Cron event sources according to the CronSpec, and in principle we try to maintain the consistency of the relevant parameters. For more information, see the [Cron binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#spec-metadata-fields).

{{% /alert %}}

| Field                 | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **schedule** *string* | Refer to [Schedule format](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#schedule-format) for a valid schedule format, for example, `@every 15m`. |

