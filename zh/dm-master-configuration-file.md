---
title: DM-master 配置文件介绍
category: reference
---

# DM-master 配置文件介绍

本文介绍 DM-master 的配置文件，包括配置文件示例与配置项说明。

## 配置文件示例

DM-master 的示例配置文件如下所示：

```toml

name = "dm-master"

# log configuration
log-level = "info"
log-file = "dm-master.log"

# DM-master listening address
master-addr = ":8261"
advertise-addr = "127.0.0.1:8261"

# URLs for peer traffic
peer-urls = "http://127.0.0.1:8291"
advertise-peer-urls = "http://127.0.0.1:8291"

# cluster configuration
initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
join = ""

# rpc configuration
rpc-timeout = "30s"
rpc-rate-burst = 40
rpc-rate-limit = 10.0

```

## 配置项说明

### Global 配置

| 配置项        | 说明                                    |
| :------------ | :--------------------------------------- |
| `name` | 标识一个 DM-master。|
| `log-level` | 日志级别：debug、info、warn、error、fatal。默认为 info。|
| `log-file` | 日志文件，如果不配置，日志会输出到标准输出中。|
| `master-addr` | DM-master 服务的地址，可以省略 IP 信息，例如：":8261"。|
| `advertise-addr` | DM-master 向外界宣告的地址。|
| `peer-urls` | DM-master 节点的对等 URL。|
| `advertise-peer-urls` | DM-master 向外界宣告的对等 URL。默认为 `peer-urls` 的值。|
| `initial-cluster` | 初始集群中所有DM-master的 `advertise-peer-urls` 值。|
| `join` | 集群里已有的DM-master的 `advertise-peer-urls` 值。加入新 DM-master 节点时使用 `join` 替代 `initial-cluster` 。|
| `rpc-timeout` | rpc超时时间，正数。使用golang标准时间单位 ns, us, ms, s, m, h。默认为 "10m"。|
| `rpc-rate-burst` | 令牌桶的大小，正数。默认为 40 。|
| `rpc-rate-limit` | 控制事件发生的频率。令牌桶最初是满的，以每秒 `rpc-rate-limit` 个令牌的速率填充令牌桶。float64类型，默认为 10.0 ，如果恰好为整数记得添加小数点和0。|
