---
title: 上游数据库配置文件介绍
aliases: ['/docs-cn/tidb-data-migration/dev/source-configuration-file/']
---

# 上游数据库配置文件介绍

本文介绍上游数据库的配置文件，包括配置文件示例与配置项说明。

## 配置文件示例

上游数据库的示例配置文件如下所示：

```yaml
source-id: "mysql-replica-01"

# 是否开启 GTID
enable-gtid: false

# 是否开启 relay log
enable-relay: false
relay-binlog-name: '' # 拉取上游 binlog 的起始文件名
relay-binlog-gtid: '' # 拉取上游 binlog 的起始 GTID

from:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "ZqMLjZ2j5khNelDEfDoUhkD5aV5fIJOe0fiog9w=" # 推荐使用 dmctl 对上游数据库的用户密码加密之后的密码
  security:                       # 上游数据库 TLS 相关配置                             
    ssl-ca: "/path/to/ca.pem"
    ssl-cert: "/path/to/cert.pem"
    ssl-key: "/path/to/key.pem"

# purge:
#   interval: 3600
#   expires: 0
#   remain-space: 15
```

> **注意：**
>
> 在 DM v2.0.1 版本中，请勿同时配置 `enable-gtid` 与 `enable-relay` 为 `true`，否则可能引发增量数据丢失问题。

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `source-id` | 标识一个 MySQL 实例。|
| `enable-gtid` | 是否使用 GTID 方式从上游拉取 binlog，默认值为 false。一般情况下不需要手动配置，如果上游数据库启用了 GTID 支持，且需要做主从切换，则将该配置项设置为 true。 |
| `enable-relay` | 是否开启 relay log，默认值为 false。 |
| `relay-binlog-name` | 拉取上游 binlog 的起始文件名，例如 "mysql-bin.000002"，该配置在 `enable-gtid` 为 false 的情况下生效。如果不配置该项，DM-worker 将从最新时间点的 binlog 文件开始拉取 binlog，一般情况下不需要手动配置。 |
| `relay-binlog-gtid` | 拉取上游 binlog 的起始 GTID，例如 "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849"，该配置在 `enable-gtid` 为 true 的情况下生效。如果不配置该项，DM-worker 将从最新时间点的 binlog GTID 开始拉取 binlog，一般情况下不需要手动配置。 |
| `host` | 上游数据库的 host。|
| `port` | 上游数据库的端口。|
| `user` | 上游数据库使用的用户名。|
| `password` | 上游数据库的用户密码。注意：推荐使用 dmctl 加密后的密码。|
| `security` | 上游数据库 TLS 相关配置。|

### relay log 清理策略配置（purge 配置项）

一般情况下不需要手动配置，如果 relay log 数据量较大，磁盘空间不足，则可以通过设置该配置项来避免 relay log 写满磁盘。

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `interval` | 定期检查 relay log 是否过期的间隔时间，默认值：3600，单位：秒。 |
| `expires` | relay log 的过期时间，默认值为 0，单位：小时。未由 relay 处理单元进行写入、或已有数据迁移任务当前或未来不需要读取的 relay log 在超过过期时间后会被 DM 删除。如果不设置则 DM 不会自动清理过期的 relay log。 |
| `remain-space` | 设置最小的可用磁盘空间。当磁盘可用空间小于这个值时，DM-worker 会尝试删除 relay log，默认值：15，单位：GB。 |

> **注意：**
>
> 仅在 `interval` 不为 0 且 `expires` 和 `remain-space` 两个配置项中至少有一个不为 0 的情况下 DM 的自动清理策略才会生效。
