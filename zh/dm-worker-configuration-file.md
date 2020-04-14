---
title: DM-worker 配置文件介绍
category: reference
---

# DM-worker 配置文件介绍

本文档主要介绍 DM-worker 的基础配置文件。在一般场景中，用户只需要使用基础配置即可完成 DM-worker 的部署。

完整配置项参考 [DM-worker 完整配置说明](dm-worker-configuration-file-full.md)。

## 配置文件示例

```toml
# Worker Configuration.

# Log configuration.
log-level = "info"
log-file = "dm-worker.log"

# DM-worker listen address.
name = "worker1"
worker-addr = ":8262"
advertise-addr = "127.0.0.1:8262"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `log-level` | 日志级别：debug、info、warn、error、fatal。默认为 info。   |
| `log-file`   | 日志文件，如果不配置日志会输出到标准输出中。   |
| `name`   | 标识一个worker。   |
| `worker-addr` | DM-worker 服务的地址，可以省略 IP 信息，例如：":8262"。|
| `advertise-addr` | DM-worker 向外界宣告的地址。 |
| `join` | 对应一个或多个 DM-master 配置中的 `master-addr`。 |

> **注意：**
>
> 以上配置为部署 DM-worker 的基础配置，一般情况下使用基础配置即可，更多配置项参考 [DM-worker 完整配置说明](dm-worker-configuration-file-full.md)。
