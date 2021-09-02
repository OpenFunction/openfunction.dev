---
title: "Cron"
linkTitle: "Cron"
weight: 25
description: >
    Event source specification of Cron
---

### CronSpec

*Belong to [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> The EventSource generates [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#component-format) for adapting Cron event sources according to the CronSpec, and in principle we try to maintain the consistency of the relevant parameters. You can get more information by visiting the [Cron binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#spec-metadata-fields).

| Field                 | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **schedule** *string* | Refer to [Schedule format](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#schedule-format) for a valid schedule format, e.g. `@every 15m` |

