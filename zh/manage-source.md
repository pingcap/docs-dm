---
title: 管理上游 MySQL 实例
summary: 了解如何管理上游 MySQL 实例。
category: reference
---

# 加密数据库密码

在 DM 相关配置文件中，如果上游 MySQL 密码不为空，要求必须使用经 dmctl 加密后的密码，否则会报错。对于同一个原始密码，每次加密后密码不同。

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 123456
```

```
VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=
```

# 加载数据源配置

`operate-source` 命令用于将数据源配置加载到 DM 集群中。

{{< copyable "" >}}

```bash
help operate-source
```

```
create/update/stop upstream MySQL/MariaDB source

Usage:
  dmctl operate-source <operate-type> <config-file> [flags]

Flags:
  -h, --help   help for operate-source

Global Flags:
  -s, --source strings   MySQL Source ID
```

## 命令用法示例

{{< copyable "" >}}

```bash
operate-source create ./source.toml
```

## 参数解释

+ `create`：创建一个上游的数据库源

+ `update`：更新一个上游的数据库源

+ `stop`：停止一个上游的数据库源

+ `config-file`：
    - 必选
    - 指定 `source.toml` 的文件路径

## 返回结果示例

{{< copyable "" >}}

```bash
operate-source create ./source.toml
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
