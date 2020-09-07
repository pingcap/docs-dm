---
title: 上游数据库配置文件介绍
---

# 上游数据库配置文件介绍

本文介绍上游数据库的配置文件，包括配置文件示例与配置项说明。

## 配置文件示例

上游数据库的示例配置文件如下所示：

```yaml
source-id: "mysql-replica-01"

# 是否开启 GTID
enable-gtid: false

from:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "ZqMLjZ2j5khNelDEfDoUhkD5aV5fIJOe0fiog9w=" # 推荐使用 dmctl 对上游数据库的用户密码加密之后的密码
  security:                       # 上游数据库 TLS 相关配置                             
    ssl-ca: "/path/to/ca.pem"
    ssl-cert: "/path/to/cert.pem"
    ssl-key: "/path/to/key.pem"
```

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `source-id` | 标识一个 MySQL 实例。|
| `enable-gtid` | 是否使用 GTID 方式从上游拉取 binlog，默认值为 false。一般情况下不需要手动配置，如果上游数据库启用了 GTID 支持，且需要做主从切换，则将该配置项设置为 true。 |
| `host` | 上游数据库的 host。|
| `port` | 上游数据库的端口。|
| `user` | 上游数据库使用的用户名。|
| `password` | 上游数据库的用户密码。注意：推荐使用 dmctl 加密后的密码。|
| `security` | 上游数据库 TLS 相关配置。|
