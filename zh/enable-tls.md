---
title: 为 DM 的连接开启加密传输
Summary: 了解如何为 DM 的连接开启加密传输
---

# 为 DM 的连接开启加密传输

本文介绍如何为 DM 的连接开启加密传输，包括 DM-master，DM-worker，dmctl 组件之间的连接以及 DM 组件与上下游数据库之间的连接。

## 为 DM-master，DM-worker，dmctl 组件之间的连接开启加密传输

### 配置开启加密传输

1. 准备证书。

    推荐为 DM-master、DM-worker 分别准备一个 Server 证书，并保证可以相互验证，而 dmctl 工具则可选择共用 Client 证书。

    有多种工具可以生成自签名证书，如 `openssl`，`easy-rsa`，`cfssl`。

    这里提供一个使用 `openssl` 生成证书的示例：[生成自签名证书](https://docs.pingcap.com/zh/tidb/stable/generate-self-signed-certificates)。

2. 配置证书。

    - DM-master

        在 DM-master 配置文件或命令行参数中设置：

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/cert.pem"
        ssl-key = "/path/to/key.pem"
        ```

    - DM-worker

        在 DM-worker 配置文件或命令行参数中设置：

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/cert.pem"
        ssl-key = "/path/to/key.pem"
        ```

    - dmctl
    
        若 DM 集群各个组件间开启加密传输后，在使用 dmctl 工具连接集群时，需要指定 client 证书，示例如下：

        {{< copyable "shell-regular" >}}

        ```bash
        ./dmctl -master-addr=127.0.0.1:8261 --ssl-ca /path/to/client-ca.pem --ssl-cert /path/to/client-cert.pem --ssl-key /path/to/client-key.pem
        ```

### 认证组件调用者身份

通常被调用者除了校验调用者提供的密钥、证书和 CA 有效性外，还需要校验调用方身份以防止拥有有效证书的非法访问者进行访问（例如：DM-worker 只能被 DM-master 访问，需阻止拥有合法证书但非 DM-master 的其他访问者访问 DM-worker）。

如希望进行组件调用者身份认证，需要在生证书时通过 `Common Name` 标识证书使用者身份，并在被调用者配置检查证书 `Common Name` 列表来检查调用者身份。

- DM-master

    在 `config` 文件或命令行参数中设置：

    ```toml
    cert-allowed-cn = ["dm"] 
    ```

- DM-worker

    在 `config` 文件或命令行参数中设置：

    ```toml
    cert-allowed-cn = ["dm"] 
    ```

### 证书重加载

DM-master、DM-worker 和 dmctl 都会在每次新建相互通讯的连接时重新读取当前的证书和密钥文件内容，实现证书和密钥的重加载。目前暂不支持 CA 的重加载。

## DM 组件与上下游数据库之间的连接开启加密传输

### 为上游数据库连接开启加密传输

1. 配置上游数据库，启用加密连接支持并设置 server 证书，具体可参考 [Using encrypted connections](https://dev.mysql.com/doc/refman/5.7/en/using-encrypted-connections.html)

2. 在 source 配置文件中设置 client 证书：

    ```yaml
    from:
        security:
            ssl-ca: "/path/to/client-ca.pem"
            ssl-cert: "/path/to/client-cert.pem"
            ssl-key: "/path/to/client-key.pem"
    ```

### 为下游 TiDB 连接开启加密传输

1. 配置下游 TiDB 启用加密连接支持，具体可参考 [配置 TiDB 启用加密连接支持](https://docs.pingcap.com/zh/tidb/stable/enable-tls-between-clients-and-servers#%E9%85%8D%E7%BD%AE-tidb-%E5%90%AF%E7%94%A8%E5%8A%A0%E5%AF%86%E8%BF%9E%E6%8E%A5%E6%94%AF%E6%8C%81)

2. 在 task 配置文件中设置 client 证书：

    ```yaml
    target-database:
        security:
            ssl-ca: "/path/to/client-ca.pem"
            ssl-cert: "/path/to/client-cert.pem"
            ssl-key: "/path/to/client-key.pem"
    ```
