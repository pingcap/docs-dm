---
title: 管理上游 MySQL 实例
summary: 了解如何管理上游 MySQL 实例。
---

# 管理上游 MySQL 实例

本文介绍了如何使用 [dmctl](dmctl-introduction.md) 组件来加密数据库密码和管理数据源配置。

## 加密数据库密码

在 DM 相关配置文件中，推荐使用经 dmctl 加密后的密码。对于同一个原始密码，每次加密后密码不同。

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 'abc!@#123'
```

```
MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
```

## 加载数据源配置

`operate-source` 命令用于将数据源配置加载到 DM 集群中。

{{< copyable "" >}}

```bash
help operate-source
```

```
create/update/stop/show upstream MySQL/MariaDB source

Usage:
  dmctl operate-source <operate-type> [config-file ...] [--print-sample-config] [flags]

Flags:
  -h, --help                  help for operate-source
  -p, --print-sample-config   print sample config file of source

Global Flags:
  -s, --source strings   MySQL Source ID
```

## 命令用法示例

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

其中 `source.toml` 的配置参考[上游数据库配置文件介绍](source-configuration-file.md)。

## 参数解释

+ `create`：创建一个或多个上游的数据库源。创建多个数据源失败时，会尝试回滚到执行命令之前的状态

+ `update`：更新一个上游的数据库源

+ `stop`：停止一个或多个上游的数据库源。停止多个数据源失败时，可能有部分数据源已成功停止

+ `show`：显示已添加的数据源以及对应的 DM-worker

+ `config-file`：
    - 指定 `source.yaml` 的文件路径
    - 可传递多个文件路径
    
+ `--print-sample-config`：打印示例配置文件。该参数会忽视其余参数

## 返回结果示例

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

## 查看 DM-master 中生效参数

{{< copyable "" >}}

```bash
get-config source mysql-replica-01
```

```
{
    "result": true,
    "msg": "",
    "cfg": "enable-gtid: false\nauto-fix-gtid: false\nrelay-dir: \"\"\nmeta-dir: \"\"\nflavor: mysql\ncharset: \"\"\nenable-relay: false\nrelay-binlog-name: \"\"\nrelay-binlog-gtid: \"\"\nsource-id: mysql-replica-01\nfrom:\n  host: 172.16.4.222\n  port: 3306\n  user: dm_user\n  password: '******'\n  max-allowed-packet: null\n  session: {}\n  security: null\npurge:\n  interval: 3600\n  expires: 0\n  remain-space: 15\nchecker:\n  check-enable: true\n  backoff-rollback: 5m0s\n  backoff-max: 5m0s\n  check-interval: 5s\n  backoff-min: 1s\n  backoff-jitter: true\n  backoff-factor: 2\nserver-id: 429595182\ntracer: {}\n"
}
```

> **注意：**
>
> `get-config` 命令仅在 DM v2.0.1 及其以后版本支持。
