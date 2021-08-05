---
title: Create a Data Source
summary: Learn how to create a data source for Data Migration (DM).
---

# Create a Data Source

> **Note:**
>
> Before creating a data source, you need to [Deploy a DM Cluster Using TiUP](deploy-a-dm-cluster-using-tiup.md)。

The document describes how to create a data source for the data migration task of TiDB Data Migration (DM).

A data source contains the data source access information. Before creating a data migration task, you need to create the data source of a task. That is because a data migration task requires quoting its corresponding data source to obtain the configuration information of access. For specific data source management commands, refer to [Manage Data Source Configurations](manage-source.md).

## Step 1: Configure the data source

1. (optional) Encrypt the data source password

    In DM configuration files, it is recommended to use the password encrypted with dmctl. You can follow the example below to obtain the encrypted password of the data source, which can be used to write the configuration file later.

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl encrypt 'abc!@#123'
    ```

    ```
    MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
    ```

2. Write the configuration file of the data source

    For each data source, you need to create an independent configuration file to create it. You can follow the example below to create a data source whose ID is "mysql-01". First create the configuration file `./source-mysql-01.yaml`：

    ```yaml
    source-id: "mysql-01"    # The ID of the data source, you can 数据源对象 ID，在数据迁移任务配置和 dmctl 命令行中引用该 source-id 可以关联到对应的数据源对象
    
    from:
      host: "127.0.0.1"
      port: 3306
      user: "root"
      password: "MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=" # 推荐使用 dmctl 对上游数据源的用户密码加密之后的密码
      security:                                        # 上游数据源 TLS 相关配置。如果没有需要则可以删除
        ssl-ca: "/path/to/ca.pem"
        ssl-cert: "/path/to/cert.pem"
        ssl-key: "/path/to/key.pem"
    ```

## 第二步：创建数据源对象

使用如下命令创建数据源对象：

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr <master-addr> operate-source create ./source-mysql-01.yaml
```

数据源配置文件的其他配置参考[数据源配置文件介绍](source-configuration-file.md)。

命令返回结果如下：

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

## 第三步：查询创建的数据源对象

创建数据源对象后，可以使用如下命令查看创建的数据源对象：

- 如果知道数据源的 `source-id`，可以通过 `dmctl get-config source <source-id>` 命令直接查看数据源配置：

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr <master-addr> get-config source mysql-01
    ```
    
    ```
    {
      "result": true,
      "msg": "",
      "cfg": "enable-gtid: false
        flavor: mysql
        source-id: mysql-01
        from:
          host: 127.0.0.1
          port: 3306
          user: root
          password: '******'
    }
    ```

- 如果不知道数据源的 `source-id`，可以先通过 `dmctl operate-source show` 命令查看源数据库列表，从中可以找到对应的数据源对象。

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr <master-addr> operate-source show
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "source is added but there is no free worker to bound",
                "source": "mysql-02",
                "worker": ""
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-01",
                "worker": "dm-worker-1"
            }
        ]
    }
    ```
