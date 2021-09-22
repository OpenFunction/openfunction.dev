---
title: "Redis 参数说明"
linkTitle: "Redis 参数说明"
weight: 15
description: >
    Redis 事件源参数说明
---

### RedisSpec

*从属 [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> EventSource 会根据 RedisSpec 生成用于适配 Redis 事件源的 [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/redis/#component-format) ，原则上我们会尽量维持相关参数的一致性。你可以通过访问 [Redis binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/redis/#spec-metadata-fields) 获取更多信息。

| 字段                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| **redisHost** *string*             | Redis 服务器的地址，如：`localhost:6379`                     |
| **redisPassword** *string*         | Redis 服务器的密码，如：`123456`                             |
| **enableTLS** *bool*               | *（可选）* 是否启用 TLS 访问，默认为 `false` 。可选：`true`，`false` |
| **failover** *bool*                | *（可选）* 是否启用 failover 特性。需要设置 sentinalMasterName 。默认为 `false` 。可选：`true`，`false` |
| **sentinelMasterName** *string*    | *（可选）* sentinel master  的名称。参考 [Redis Sentinel Documentation](https://redis.io/topics/sentinel) 。 |
| **redeliverInterval** *string*     | *（可选）* 重新发送的时间间隔。默认为 `60s`。`0` 表示禁用重新发送机制。如：`30s` |
| **processingTimeout** *string*     | *（可选）* 消息处理超时时间。默认为 `15s`。`0` 表示禁用超时。如：`30s` |
| **redisType** *string*             | *（可选）* Redis 的类型。可选：单节点模式的 `node` ，集群模式的 `cluster`。默认为 `node` |
| **redisDB** *int64*                | *（可选）* 连接到 Redis 的数据库索引值。仅在 redisType 为 `node` 时生效。默认为 `0` 。 |
| **redisMaxRetries** *int64*        | *（可选）* 最大重试次数。默认不重试。如：`5`                 |
| **redisMinRetryInterval** *string* | *（可选）* 重试的最小退避时间。默认值是 `8ms`。`-1` 表示禁用退避时间。如：`10ms` |
| **redisMaxRetryInterval** *string* | *（可选）* 重试的最大退避时间。默认值是 `512ms`。`-1` 表示禁用退避时间。如：`5s` |
| **dialTimeout** *string*           | *（可选）* 建立新连接的超时时间。默认为 `5s`。               |
| **readTimeout** *string*           | *（可选）* 读取超时时间。如果超时则会导致 Redis 命令失败而非以阻塞方式等待。默认为 `3s`，`-1` 表示没有超时。 |
| **writeTimeout** *string*          | *（可选）* 写入超时时间。如果超时则会导致 Redis 命令失败而非以阻塞方式等待。默认与 readTimeout 一致。 |
| **poolSize** *int64*               | *（可选）* 最大连接数量。默认每个物理 CPU 负载 10 个连接。如：`20` |
| **poolTimeout** *string*           | *（可选）* 连接池的超时时间。默认是 readTimeout + 1 秒。如：`50s` |
| **maxConnAge** *string*            | *（可选）* 连接老化时间。默认不关闭老化的连接。如：`30m`     |
| **minIdleConns** *int64*           | *（可选）* 维持的最小空闲连接数，以避免创建新连接带来的性能下降。默认为 `0`。如：`2` |
| **idleCheckFrequency** *string*    | *（可选）* 空闲连接回收器的检查频率。默认为 `1m`。`-1` 表示禁用空闲连接回收器。如：`-1` |
| **idleTimeout** *string*           | *（可选）* 关闭客户端闲置连接的超时时间。应该小于服务器的超时时间。默认为 `5m`。`-1` 表示禁用空闲超时检查。如：`10m` |
