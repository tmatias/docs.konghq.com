---
title: Rate Limiting Advanced Plugin
---
# Rate Limiting Advanced Plugin

Rate limiting functionality is available in two plugins - one is
[bundled with Kong Community Edition (CE)](/plugins/rate-limiting/),
and the other is bundled with Kong Enterprise Edition (EE). This page documents the Kong EE version of
the Rate Limiting plugin, which has greater functionality and performance than the CE version.

## Terminology

- `plugin`: a plugin executing actions inside Kong before or after a request has been proxied to the upstream API.
- `Service`: the Kong entity representing an external upstream API or microservice.
- `Route`: the Kong entity representing a way to map downstream requests to upstream services.
- `Consumer`: the Kong entity representing a developer or machine using the API. When using Kong, a Consumer only communicates with Kong which proxies every call to the said upstream API.
- `Credential`: a unique string associated with a Consumer, also referred to as an API key.
- `upstream service`: this refers to your own API/service sitting behind Kong, to which client requests are forwarded.
- `API`: a legacy entity used to represent your upstream services. Deprecated in favor of Services since 0.13.0.


## Configuration

Method 1: apply it on top of an API by executing the following request on your Kong server:

```
$ curl -i -X POST http://kong:8001/apis/{api}/plugins \
  --data name=rate-limiting-advanced \
  --data config.limit=10 \
  --data config.sync_rate=10 \
  --data config.window_size=60
```

Method 2: apply it globally (on all APIs) by executing the following request on your Kong server:

```
$ curl -i -X POST http://kong:8001/plugins \
  --data name=rate-limiting-advanced \
  --data config.limit=10 \
  --data config.sync_rate=10 \
  --data config.window_size=60
```
## Configuration Parameters

| Form Parameter | default | description
|----------------|---------|-------------
| `name`|| The name of the plugin to use, in this case: `rate-limiting-advanced`.
| `service_id` || The id of the Service which this plugin will target.
| `route_id` || The id of the Route which this plugin will target.
| `consumer_id` || The id of the Consumer which this plugin will target.
| `api_id` || The id of the API which this plugin will target. Note: The API Entity is deprecated since Kong 0.13.0.
|`config.limit`|| one of more request per window to apply
|`config.window_size`||One more more window sizes to apply (defined in seconds).
|`config.identifier` | `consumer` | How to define the rate limit key. Can be `ip`, `credential`, or `consumer`.
|`config.dictionary_name`| <div style="max-width: 160px;">`kong_rate_limiting_counters`</div> | The shared dictionary where counters will be stored until the next sync cycle.
|`config.sync_rate` | | How often to sync counter data to the central data store. A value of 0 results in synchronous behavior; a value of -1 ignores sync behavior entirely and only stores counters in node memory. A value greater than 0 will sync the counters in that many number of seconds.
|`config.namespace(optional)`| `random string`|The rate limiting library namespace to use for this plugin instance. Counter data and sync configuration is shared in a namespace.
|`config.strategy`| `cluster` | The sync strategy to use; `cluster` and `redis` are supported.
|`config.redis.host(semi-optional)` | | Host to use for Redis connection when the `redis` strategy is defined.
|`config.redis.port(semi-optional)`||Port to use for Redis connection when the `redis` strategy is defined.
|`config.redis.timeout(semi-optional)`| | Connection timeout to use for Redis connection when the `redis` strategy is defined.
|`config.redis.password(semi-optional)`|| Password to use for Redis connection when the `redis` strategy is defined. If undefined, no AUTH commands are sent to Redis.
|`config.redis.database(semi-optional)`|`0`|Database to use for Redis connection when the redis strategy is defined.
|`config.redis.sentinel_master(semi-optional)`||Sentinel master to use for Redis connection when the `redis` strategy is defined. Defining this value implies using Redis Sentinel.
|`config.redis.sentinel_role(semi-optional)`||Sentinel role to use for Redis connection when the `redis` strategy is defined. Defining this value implies using Redis Sentinel.
|`config.redis.sentinel_addresses(semi-optional)`||Sentinel addresses to use for Redis connection when the `redis` strategy is defined. Defining this value implies using Redis Sentinel.
|`config.window_type`| `sliding` | This sets the time window to either `sliding` or `fixed`. Sliding windows factor in weighted request counts from previous time segments when determining whether a client has exceeded its limit, whereas fixed windows only count requests during the current time segment.
|`config.hide_client_headers`| `false` | Controls whether `X-Ratelimit-Remaining` and `X-Ratelimit-Limit` headers are sent to clients. When set to `true`, headers are not sent/are hidden.

**Note:  Redis configuration values are ignored if the "cluster" strategy is used.**

**Note: PostgreSQL 9.5+ is required when using the "cluster" strategy with "postgres" as the backing Kong
cluster data store. This requirement varies from the PostgreSQL 9.4+ requirement as described
in the <a href="/install/source">Kong Community Edition documentation</a>.**

**Note: The `dictionary_name` directive was added to prevent the usage of the `kong` shared dictionary, which could lead to `no memory` errors**

---

## Notes
An arbitrary number of limits/window sizes can be applied per plugin instance. This allows users to create multiple rate limiting windows (e.g., rate limit per minute and per hour, and/or per any arbitrary window size); because of limitation with Kong's plugin configuration interface, each *nth* limit will apply to each *nth* window size. For example:

```
$ curl -i -X POST http://kong:8001/apis/{api}/plugins \
  --data name=rate-limiting-advanced \
  --data config.limit=10,100 \
  --data config.window_size=60,3600 \
  --data config.sync_rate=10
```
This will apply rate limiting policies, one of which will trip when 10 hits have been counted in 60 seconds, or when 100 hits have been counted in 3600 seconds.
