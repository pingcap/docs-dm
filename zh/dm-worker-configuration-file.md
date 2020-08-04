---
title: DM-worker 配置文件介绍
aliases: ['/docs-cn/tidb-data-migration/dev/dm-worker-configuration-file/']
---

# DM-worker 配置文件介绍

本文介绍 DM-worker 的配置文件，包括配置文件示例与配置项说明。

## 配置文件示例

```toml
# Worker Configuration.

name = "worker1"

# Log configuration.
log-level = "info"
log-file = "dm-worker.log"

# DM-worker listen address.
worker-addr = ":8262"
advertise-addr = "127.0.0.1:8262"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `name`   | 标识一个 DM-worker。   |
| `log-level` | 日志级别：debug、info、warn、error、fatal。默认为 info。   |
| `log-file`   | 日志文件，如果不配置日志会输出到标准输出中。   |
| `worker-addr` | DM-worker 服务的地址，可以省略 IP 信息，例如：":8262"。|
| `advertise-addr` | DM-worker 向外界宣告的地址。 |
| `join` | 对应一个或多个 DM-master 配置中的 [`master-addr`](dm-master-configuration-file.md#global-配置)。 |
