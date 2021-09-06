---
title: "Cron 参数说明"
linkTitle: "Cron 参数说明"
weight: 25
description: >
    Cron 事件源参数说明
---

### CronSpec

*从属 [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> EventSource 会根据 CronSpec 生成用于适配 Cron 事件源的 [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#component-format) ，原则上我们会尽量维持相关参数的一致性。你可以通过访问 [Cron binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#spec-metadata-fields) 获取更多信息。

| 字段                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| **schedule** *string* | 参考 [Schedule format](https://docs.dapr.io/reference/components-reference/supported-bindings/cron/#schedule-format) 了解合法的 schedule 格式。如：`@every 15m` |

