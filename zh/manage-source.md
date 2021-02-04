---
title: 管理上游 MySQL 实例
summary: 了解如何管理上游 MySQL 实例。
aliases: ['/docs-cn/tidb-data-migration/dev/manage-source/']
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

### 命令用法示例

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

其中 `source.toml` 的配置参考[上游数据库配置文件介绍](source-configuration-file.md)。

### 参数解释

+ `create`：创建一个或多个上游的数据库源。创建多个数据源失败时，会尝试回滚到执行命令之前的状态

+ `update`：更新一个上游的数据库源

+ `stop`：停止一个或多个上游的数据库源。停止多个数据源失败时，可能有部分数据源已成功停止

+ `show`：显示已添加的数据源以及对应的 DM-worker

+ `config-file`：
    - 指定 `source.yaml` 的文件路径
    - 可传递多个文件路径
    
+ `--print-sample-config`：打印示例配置文件。该参数会忽视其余参数

### 创建数据源

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

### 查看数据源配置

如果直到 source-id，可以通过 `dmctl --master-addr <master-addr> get-config source <source-name>` 命令直接查看源数据库配置

{{< copyable "" >}}

```bash
get-config source mysql-replica-01
```

```
{
  "result": true,
  "msg": "",
  "cfg": "enable-gtid: false
    flavor: mysql
    source-id: mysql-replica-01
    from:
      host: 127.0.0.1
      port: 8407
      user: root
      password: '******'
}
```

如果不知道 source-id，可以先通过 `dmctl --master-addr <master-addr> operate-source show` 查看源数据库列表

{{< copyable "" >}}

```bash
operate-source show
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "source is added but there is no free worker to bound",
            "source": "mysql-replica-02",
            "worker": ""
        },
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```