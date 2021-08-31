---
title: "Redis"
linkTitle: "Redis"
weight: 15
description: >
    Event source specification of Redis
---

### RedisSpec

*Belong to [EventSourceSpec]({{<ref "_index.md#eventsourcespec" >}})*

> The EventSource generates [Dapr Bindings Components](https://docs.dapr.io/reference/components-reference/supported-bindings/redis/#component-format) for adapting Redis event sources according to the RedisSpec, and in principle we try to maintain the consistency of the relevant parameters. You can get more information by visiting the [Redis binding spec](https://docs.dapr.io/reference/components-reference/supported-bindings/redis/#spec-metadata-fields).

| Field                              | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| **redisHost** *string*             | Address of the Redis server, e.g. `localhost:6379`           |
| **redisPassword** *string*         | Password for the Redis server, e.g. `123456`                 |
| **enableTLS** *bool*               | *(Optional)* Whether to enable TLS access, default is `false`. Optional: `true`, `false` |
| **failover** *bool*                | *(Optional)* Whether to enable the failover feature. Requires the sentinalMasterName to be set. Default is `false`. Optional: `true`, `false`. |
| **sentinelMasterName** *string*    | *(Optional)* The name of the sentinel master. Refer to [Redis Sentinel Documentation](https://redis.io/topics/sentinel). |
| **redeliverInterval** *string*     | *(Optional)* The interval for redeliver. Default is `60s`. `0` means the redeliver mechanism is disabled. E.g. `30s` |
| **processingTimeout** *string*     | *(Optional)* Message processing timeout. Default is `15s`. `0` means timeout is disabled. E.g. `30s` |
| **redisType** *string*             | *(Optional)* The type of Redis. Optional: `node` for single-node mode, `cluster` for cluster mode. Default is `node`. |
| **redisDB** *int64*                | *(Optional)* The database index to connect to Redis. Effective only if redisType is `node`. Default is `0`. |
| **redisMaxRetries** *int64*        | *(Optional)* Maximum number of retries. Default is no retries. E.g. `5` |
| **redisMinRetryInterval** *string* | *(Optional)* Minimum backoff time for retries. The default value is `8ms`. `-1` indicates that the backoff time is disabled. E.g. `10ms` |
| **redisMaxRetryInterval** *string* | *(Optional)* Maximum backoff time for retries. The default value is `512ms`. `-1` indicates that the backoff time is disabled. E.g. `5s` |
| **dialTimeout** *string*           | *(Optional)* Timeout to establish a new connection. Default is `5s`. |
| **readTimeout** *string*           | *(Optional)* Read timeout. A timeout will cause Redis commands to fail rather than wait in a blocking fashion. Default is `3s`, `-1` means disabled. |
| **writeTimeout** *string*          | *(Optional)* Write timeout. A timeout will cause Redis commands to fail rather than wait in a blocking fashion. Default is consistent with readTimeout. |
| **poolSize** *int64*               | *(Optional)* Maximum number of connections. Default is 10 connections per runtime.NumCPU. E.g. `20` |
| **poolTimeout** *string*           | *(Optional)* The timeout for the connection pool. The default is readTimeout + 1 second. E.g. `50s` |
| **maxConnAge** *string*            | *(Optional)* Connection aging time. The default is not to close the aging connection. E.g. `30m` |
| **minIdleConns** *int64*           | *(Optional)* The minimum number of idle connections to maintain to avoid performance degradation from creating new connections. Defaults to `0`. E.g. `2` |
| **idleCheckFrequency** *string*    | * (Optional) * Frequency of idle connection recycler checks. Default is `1m`. `-1` means the idle connection recycler is disabled. E.g. `-1` |
| **idleTimeout** *string*           | *(Optional)* Timeout to close idle client connections. Should be less than the server's timeout. Default is `5m`. `-1` means disable idle timeout check. E.g. `10m` |
