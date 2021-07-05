- [WebSocket Settings](#websocket-settings)
  - [Server](#server)
  - [SSL Settings](#ssl-settings)
  - [HTTP REST API](#http-rest-api)
  - [Adapters](#adapters)
- [Applications](#applications)
  - [Default Application](#default-application)
  - [Apps Manager](#apps-manager)
  - [Metrics](#metrics)
- [Rate Limiting](#rate-limiting)
  - [Events Soft Limits](#events-soft-limits)
- [Channels](#channels)
  - [Presence Channel Limits](#presence-channel-limits)
  - [Channels Soft Limits](#channels-soft-limits)
- [Databases](#databases)
  - [MySQL Configuration](#mysql-configuration)
  - [PostgreSQL Configuration](#postgresql-configuration)
  - [Redis Configuration](#redis-configuration)
- [Database Pooling](#database-pooling)
- [Debugging](#debugging)
  - [Node Metadata](#node-metadata)

## WebSocket Settings

### Server

Configuration needed to specify the protocol, port and host for the server.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `PORT` | `6001` | - | The host used for both the REST API and the WebSocket server. |

### SSL Settings

Setting one of the following variables will create a SSL version of the app.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `SSL_CERT` | `''` | - | The path for SSL certificate file. |
| `SSL_KEY` | `''` | - | The path for SSL key file. |
| `SSL_PASS` | `''` | - | The passphrase for the SSL key file. |

### HTTP REST API

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `HTTP_MAX_REQUEST_SIZE` | `100` | - | The maximum size, in MB, for the total size of the request before throwing `413 Entity Too Large`. A hard limit has been set to 100 MB. |

### Adapters

For local, single-instance applications, the default local adapter is fine.

However, Redis is needed if you plan to run on multiple instances or processes at the same time to ensure availability.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `ADAPTER_DRIVER` | `local` | `redis`, `local` | The adapter driver to use to store and retrieve each app with channels' persistent data. |
| `ADAPTER_REDIS_PREFIX` | `''` | - | The Redis adapter's Pub/Sub channels prefix. |

- `redis` - Enabled Pub/Sub communication between processes/nodes, can be scaled horizontally without issues.
- `local` - There is no communication or Pub/Sub. Recommended for single-instance, single-process apps.

## Applications

### Default Application

By default, the app is using a predefined list of applications to allow access.

In case you opt-in for another `APP_MANAGER_DRIVER`, these are the variables you can change in order to change the app settings.

For the rate limiting and max connections options, setting limits to `-1` will disable the rate limiting and/or max allowed connections.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DEFAULT_APP_ID` | `app-id` | - | The default app id for the array driver. |
| `DEFAULT_APP_KEY` | `app-key` | - | The default app key for the array driver. |
| `DEFAULT_APP_SECRET` | `app-secret` | - | The default app secret for the array driver. |
| `DEFAULT_APP_MAX_CONNS` | `-1` | - | The default app's limit of concurrent connections. |
| `DEFAULT_APP_ENABLE_CLIENT_MESSAGES` | `false` | `true`, `false` | Wether client messages should be enabled for the app. |
| `DEFAULT_APP_MAX_BACKEND_EVENTS_PER_SEC` | `-1` | - | The default app's limit of `/events` endpoint events broadcasted per second. You can [configure rate limiting database store](#rate-limiting) |
| `DEFAULT_APP_MAX_CLIENT_EVENTS_PER_SEC` | `-1` | - | The default app's limit of client events broadcasted per second, by a single socket. You can [configure rate limiting database store](#rate-limiting) |
| `DEFAULT_APP_MAX_READ_REQ_PER_SEC` | `-1` | - | The default app's limit of read endpoint calls per second. You can [configure rate limiting database store](#rate-limiting) |

### Apps Manager

The apps manager manages the allowed apps to connect to the WS and the API. Defaults to the local, `array` driver predefined by the `DEFAULT_APP_*` variables.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `APP_MANAGER_DRIVER` | `array` | `array`, `dynamodb`, `mysql`, `postgres` | The driver used to retrieve the app. |

Additionally, the following settings apply for further configurations of the app manager that was selected:

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `APP_MANAGER_DYNAMODB_TABLE` | `apps` | - | The table name to pull the data from. Only for `dynamodb`. |
| `APP_MANAGER_DYNAMODB_REGION` | `us-east-1` | - | The DynamoDB region the table was deployed in. For global tables, pick any region. Only for `dynamodb`. |
| `APP_MANAGER_DYNAMODB_ENDPOINT` | `''` | - | The endpoint to connect to DynamoDB. Optional, used for testing or for local DynamoDB configurations. Only for `dynamodb`. |
| `APP_MANAGER_MYSQL_TABLE` | `apps` | - | The table name to pull the data from. Only for `mysql`. |
| `APP_MANAGER_MYSQL_VERSION` | `8.0` | - | The MySQL version so that the Knex connector know how to connect. Only for `mysql`. |
| `APP_MANAGER_POSTGRES_TABLE` | `apps` | - | The table name to pull the data from. Only for `postgres`. |
| `APP_MANAGER_POSTGRES_VERSION` | `13.3` | - | The PostgreSQL version so that the Knex connector know how to connect. Only for `postgres`. |

To learn more about app managers for third-party databases, please consider:

- [DynamoDB](APP_MANAGERS.md#aws-dynamodb)
- [MySQL](APP_MANAGERS.md#mysql-driver)
- [PostgreSQL](APP_MANAGERS.md#postgresql-driver)

### Metrics

The metrics feature allows you to store metrics at the node level. This can easily be done under the hood with Prometheus. All you need to do is to set up your own Prometheus server and make it scrap the HTTP REST API of the each node that pWS runs on, on the `/metrics` endpoint.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `METRICS_ENABLED` | `false` | `true`, `false` | Wether to enable the metrics or not. For Prometheus, enabling it will expose a `/metrics` endpoint. |
| `METRICS_DRIVER` | `prometheus` | `prometheus` | The driver used to scrap the metrics. For now, only `prometheus` is available. Soon, Pushgateway will be available. |
| `METRICS_PROMETHEUS_PREFIX` | `pws_` | - | The prefix to add to the metrics in Prometheus to differentiate from other metrics in Prometheus. |

## Rate Limiting

Rate limiting is helping you limit the access for applications at the app level with [app settings, per se](#default-application).

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `RATE_LIMITER_DRIVER` | `local` | `local`, `redis` | The driver used for rate limiting counting. |

- `local` - Rate limiting is stored within the memory and is lost upon process exit.
- `redis` - Rate limiting is centralized in Redis using the key-value store. Recommended when having a multi-node configuration.

### Events Soft Limits

Beside the rate limiting, you can set soft limits for the incoming data, such as the maximum allowed event size or the maximum event name length.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `EVENT_MAX_CHANNELS_AT_ONCE` | `100` | - | The maximum amount of channels that the client can broadcast to from a single `/events` request. |
| `EVENT_MAX_NAME_LENGTH` | `200` | - | The maximum length of the event name that is allowed. |
| `EVENT_MAX_SIZE_IN_KB` | `100` | - | The maximum size, in KB, for the broadcasted payloads incoming from the clients. |


## Channels

### Presence Channel Limits

When dealing with presence channel, connection details must be stored within the app.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `PRESENCE_MAX_MEMBER_SIZE` | `10` | - | The maximum member size, in KB, for each member in a presence channel. |
| `PRESENCE_MAX_MEMBERS` | `100` | - | The maximum amount of members that can simultaneously be connected in a presence channel. |

### Channels Soft Limits

Beside the rate limiting, you can set soft limits for the incoming data, such as the maximum allowed event size or the maximum event name length.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `CHANNEL_MAX_NAME_LENGTH` | `100` | - | The maximum length of the channel name that is allowed. The specific-prefix names are also counted. |

## Databases

### MySQL Configuration

Configuration needed to connect to a MySQL server.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DB_MYSQL_HOST` | `127.0.0.1` | - | The MySQL host used for `mysql` driver. |
| `DB_MYSQL_PORT` | `3306` | - | The MySQL port used for `mysql` driver. |
| `DB_MYSQL_USERNAME` | `root` | - | The MySQL username used for `mysql` driver. |
| `DB_MYSQL_PASSWORD` | `password` | - | The MySQL password used for `mysql` driver. |
| `DB_MYSQL_DATABASE` | `main` | - | The MySQL database used for `mysql` driver. |

This database supports [Database Pooling](#database-pooling).

### PostgreSQL Configuration

Configuration needed to connect to a PostgreSQL server.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DB_POSTGRES_HOST` | `127.0.0.1` | - | The PostgreSQL host used for `postgres` driver. |
| `DB_POSTGRES_PORT` | `3306` | - | The PostgreSQL port used for `postgres` driver. |
| `DB_POSTGRES_USERNAME` | `root` | - | The PostgreSQL username used for `postgres` driver. |
| `DB_POSTGRES_PASSWORD` | `password` | - | The PostgreSQL password used for `postgres` driver. |
| `DB_POSTGRES_DATABASE` | `main` | - | The PostgreSQL database used for `postgres` driver. |

This database supports [Database Pooling](#database-pooling).

### Redis Configuration

Configuration needed to connect to a Redis server. The configuration is heavily based on [`ioredis`](https://github.com/luin/ioredis).
It is therefore recommended to checkout [their documentation regarding the options](https://github.com/luin/ioredis/blob/master/API.md#new-redisport-host-options).
Not all options offered by `ioredis` have been implemented. The following is a list of options made available so far:

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DB_REDIS_HOST` | `127.0.0.1` | - | The Redis host used for `redis` driver. |
| `DB_REDIS_PORT` | `6379` | - | The Redis port used for `redis` driver. |
| `DB_REDIS_DB` | `0` | - | The Redis database used for `redis` driver. |
| `DB_REDIS_USERNAME` | `null` | - | The Redis username used for authentication for `redis` driver. |
| `DB_REDIS_PASSWORD` | `null` | - | The Redis password used for authentication for `redis` driver. |
| `DB_REDIS_PREFIX` | `pws` | - | The key prefix for Redis. Only for `redis` driver. |

The following options are available when connecting to Redis through one or more Sentinels:

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DB_REDIS_SENTINELS` | `null` | - | A json encoded array of objects with `host` and `port` of Sentinels to connect to. Only for `redis` driver. |
| `DB_REDIS_SENTINEL_PASSWORD` | `null` | - | An optional password used for authentication to the Sentinels. Only for `redis` driver. |
| `DB_REDIS_INSTANCE_NAME` | `mymaster` | - | The name of the Redis instance to which a connection should be established through the configured Sentinel(s). Only for `redis` driver. |

Please be aware that configuring the Sentinel options will override some of the other options like `DB_REDIS_HOST` and `DB_REDIS_PORT`,
but not `DB_REDIS_DB` or `DB_REDIS_PASSWORD` for example. More details can be found in the [`ioredis` documentation for Redis Sentinel](https://github.com/luin/ioredis#sentinel).

## Database Pooling

Behind the scenes, the connections to the relational databases are made using [Knex](https://knexjs.org/). In case you are using Connection Pooling in your database,
you can instruct Knex to connect to them via pooling instead of a regular one connection per statement.

| Environment variable | Object dot-path | Default | Available values | Description |
| - | - | - | - | - |
| `DB_POOLING_ENABLED` | `false` | `true`, `false` | Wether to enable the database pooling. The pooling will be truly enable only if the database suppport pooling in Knex. |
| `DB_POOLING_MIN` | `0` | - | The minimum amount of connections. |
| `DB_POOLING_MAX` | `7` | - | The maximum amount of connections. |

## Debugging

Options for application debugging. Should be disabled on production environments.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `DEBUG` | `false` | `true`, `false` | Weteher the app should be in debug mode, being very verbose with what happens behind the scenes. |

### Node Metadata

Node settings include assigning identifiers for the running node.

| Environment variable | Default | Available values | Description |
| - | - | - | - |
| `NODE_ID` | random UUIDv4 string | - | An unique ID given to the node in which the process runs. Used by other features to label data. |
| `POD_ID` | `null` | - | The Pod name if the app runs in Kubernetes. Used by other features to label data. |