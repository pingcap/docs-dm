---
title: 上游数据库配置文件介绍
category: reference
---

# 上游数据库配置文件介绍

本文介绍上游数据库的配置文件，包括配置文件示例与配置项说明。

## 配置文件示例

上游数据库的示例配置文件如下所示：

```toml

source-id = "mysql-replica-01"

[from]
host = "127.0.0.1"
port = 3306
user = "root"
password = "ZqMLjZ2j5khNelDEfDoUhkD5aV5fIJOe0fiog9w=" # 使用 dmctl 对上游数据库的用户密码加密之后的密码
```

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `source-id` | 标识一个 MySQL 实例。|
| `host` | 上游数据库的 host。|
| `port` | 上游数据库的端口。|
| `user` | 上游数据库使用的用户名。|
| `password` | 连接数据库使用的密码。注意：需要使用 dmctl 加密后的密码。|
