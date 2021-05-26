---
title: 创建数据源对象
---

在创建数据源对象之前你需要先完成
1. [使用 TiUP 部署 DM 集群](deploy-a-dm-cluster-using-tiup.md)

# 创建数据源对象

数据源对象包含了访问数据源的信息。在创建数据迁移任务之前，需要先创建任务的数据源对象。数据迁移任务需要引用对应的数据源对象来获取访问配置信息。详细的数据源管理命令请参考[管理上游数据源](manage-source.md)。

## step 1 配置数据源

### step 1.1  加密数据源密码【可选】

在 DM 的配置文件中，推荐使用经 dmctl 加密后的密文密码。按照下面的示例可以获得数据源的密文密码，将用于 step 1.2 的数据源配置。

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 'abc!@#123'
```

```
MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
```

### step 1.2 编写数据源配置文件

每个数据源需要一个单独的配置文件来创建数据源对象。按照下面示例创建 ID 为 "mysql-01" 的数据源对象，创建数据源配置文件 `./source-mysql-01.yaml`：

```yaml
source-id: "mysql-01"    # 数据源对象 ID，在数据迁移任务配置和 dmctl 命令行中引用该 source-id 可以关联到对应的数据源对象

from:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=" # 推荐使用 dmctl 对上游数据库的用户密码加密之后的密码
  security:                                        # 上游数据库 TLS 相关配置。如果没有需要则可以删除
    ssl-ca: "/path/to/ca.pem"
    ssl-cert: "/path/to/cert.pem"
    ssl-key: "/path/to/key.pem"
```

## step 2 创建数据源对象

使用如下命令创建数据源对象：

{{< copyable "" >}}

```bash
tiup dmctl operate-source create ./source-mysql-01.yaml --master-addr <master-addr>
```

其中 `source.yaml` 的配置参考[数据源配置文件介绍](source-configuration-file.md)。

结果如下：

{{< copyable "" >}}

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

## step 3 查询创建的数据源对象

创建数据源对象后，可以使用下面的命令查看创建的数据源对象。

如果知道 source-id，可以通过 `dmctl get-config source <source-id>` 命令直接查看数据源配置。

{{< copyable "" >}}

```bash
tiup dmctl get-config source mysql-replica-01 --master-addr <master-addr>
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
      port: 3306
      user: root
      password: '******'
}
```

如果不知道 source-id，可以先通过 `dmctl operate-source show` 查看源数据库列表。

{{< copyable "" >}}

```bash
tiup dmctl operate-source show --master-addr <master-addr>
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
