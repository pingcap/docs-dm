---
title: 使用 TiUP 部署 DM 集群
summary: 学习如何使用 TiUP DM 组件来部署 TiDB Data Migration 工具。
aliases: ['/docs-cn/tidb-data-migration/dev/deploy-a-dm-cluster-using-ansible/','/zh/tidb-data-migration/dev/deploy-a-dm-cluster-using-ansible']
---

# 使用 TiUP 部署 DM 集群

[TiUP](https://github.com/pingcap/tiup) 是 TiDB 4.0 版本引入的集群运维工具，[TiUP DM](maintain-dm-using-tiup.md) 是 TiUP 提供的使用 Golang 编写的集群管理组件，通过 TiUP DM 组件就可以进行日常的运维工作，包括部署、启动、关闭、销毁、扩缩容、升级 DM 集群以及管理 DM 集群参数。

目前 TiUP 可以支持部署 v2.0 及以上版本的 DM。本文将介绍不同集群拓扑的具体部署步骤。

> **注意：**
>
> 如果部署机器的操作系统支持 SELinux，请确保 SELinux 处于关闭状态。

## 第 1 步：在中控机上安装 TiUP 组件

使用普通用户登录中控机，以 `tidb` 用户为例，后续安装 TiUP 及集群管理操作均通过该用户完成：

1. 执行如下命令安装 TiUP 工具：

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

2. 按如下步骤设置 TiUP 环境变量：

    重新声明全局环境变量：

    {{< copyable "shell-regular" >}}

    ```shell
    source .bash_profile
    ```

    确认 TiUP 工具是否安装：

    {{< copyable "shell-regular" >}}

    ```shell
    which tiup
    ```

3. 安装 TiUP DM 组件

    {{< copyable "shell-regular" >}}

    ```shell
    tiup install dm
    ```

4. 如果已经安装，则更新 TiUP DM 组件至最新版本：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update --self && tiup update dm
    ```

    预期输出 `Update successfully!` 字样。

## 第 2 步：编辑初始化配置文件

请根据不同的集群拓扑，编辑 TiUP 所需的集群初始化配置文件。

请根据[配置文件模板](https://github.com/pingcap/tiup/blob/master/examples/dm/topology.example.yaml)，新建一个配置文件 `topology.yaml`。如果有其他组合场景的需求，请根据多个模板自行调整。

部署 3 个 DM-master、3 个 DM-worker 与 1 个监控组件的配置如下：

```yaml
---
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/home/tidb/dm/deploy"
  data_dir: "/home/tidb/dm/data"
  # arch: "amd64"

master_servers:
  - host: 172.19.0.101
  - host: 172.19.0.102
  - host: 172.19.0.103

worker_servers:
  - host: 172.19.0.101
  - host: 172.19.0.102
  - host: 172.19.0.103

monitoring_servers:
  - host: 172.19.0.101

grafana_servers:
  - host: 172.19.0.101

alertmanager_servers:
  - host: 172.19.0.101
```

> **注意：**
>
> - 如果不需要确保 DM 集群高可用，则可只部署 1 个 DM-master 节点，且部署的 DM-worker 节点数量不少于上游待迁移的 MySQL/MariaDB 实例数。
>
> - 如果需要确保 DM 集群高可用，则推荐部署 3 个 DM-master 节点，且部署的 DM-worker 节点数量大于上游待迁移的 MySQL/MariaDB 实例数（如 DM-worker 节点数量比上游实例数多 2 个）。
>
> - 对于需要全局生效的参数，请在配置文件中 `server_configs` 的对应组件下配置。
>
> - 对于需要某个节点生效的参数，请在具体节点的 `config` 中配置。
>
> - 配置的层次结构使用 `.` 表示。如：`log.slow-threshold`。更多格式参考 [TiUP 配置参数模版](https://github.com/pingcap/tiup/blob/master/examples/dm/topology.example.yaml)。
>
> - 更多参数说明，请参考 [master `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/master/dm-master.toml)、[worker `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/worker/dm-worker.toml)
>
> - 需要确保以下组件间端口可正常连通：
>
>     - 各 DM-master 节点间的 `peer_port`（默认为 `8291`）可互相连通。
>
>     - 各 DM-master 节点可连通所有 DM-worker 节点的 `port`（默认为 `8262`）。
>
>     - 各 DM-worker 节点可连通所有 DM-master 节点的 `port`（默认为 `8261`）。
>
>     - TiUP 节点可连通所有 DM-master 节点的 `port`（默认为 `8261`）。
>
>     - TiUP 节点可连通所有 DM-worker 节点的 `port`（默认为 `8262`）。

## 第 3 步：执行部署命令

> **注意：**
>
> 通过 TiUP 进行集群部署可以使用密钥或者交互密码方式来进行安全认证：
>
> - 如果是密钥方式，可以通过 `-i` 或者 `--identity_file` 来指定密钥的路径；
> - 如果是密码方式，可以通过 `-p` 进入密码交互窗口；
> - 如果已经配置免密登录目标机，则不需填写认证。

{{< copyable "shell-regular" >}}

```shell
tiup dm deploy dm-test ${version} ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```

以上部署命令中：

- 通过 TiUP DM 部署的集群名称为 `dm-test`。
- 部署版本为 `${version}`，可以通过执行 `tiup list dm-master` 来查看 TiUP 支持的最新版本。
- 初始化配置文件为 `topology.yaml`。
- `--user root`：通过 root 用户登录到目标主机完成集群部署，该用户需要有 ssh 到目标机器的权限，并且在目标机器有 sudo 权限。也可以用其他有 ssh 和 sudo 权限的用户完成部署。
- `-i` 及 `-p`：非必选项，如果已经配置免密登录目标机，则不需填写，否则选择其一即可。`-i` 为可登录到目标机的 root 用户（或 `--user` 指定的其他用户）的私钥，也可使用 `-p` 交互式输入该用户的密码。
- TiUP DM 使用内置的 SSH 客户端，如需使用系统自带的 SSH 客户端，请参考 TiUP DM 文档中[使用中控机系统自带的 SSH 客户端连接集群](maintain-dm-using-tiup.md#使用中控机系统自带的-ssh-客户端连接集群)章节进行设置。

预期日志结尾输出会有 ```Deployed cluster `dm-test` successfully``` 关键词，表示部署成功。

## 第 4 步：查看 TiUP 管理的集群情况

{{< copyable "shell-regular" >}}

```shell
tiup dm list
```

TiUP 支持管理多个 DM 集群，该命令会输出当前通过 TiUP DM 管理的所有集群信息，包括集群名称、部署用户、版本、密钥信息等：

```log
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
dm-test  tidb  v2.0.0  /root/.tiup/storage/dm/clusters/dm-test  /root/.tiup/storage/dm/clusters/dm-test/ssh/id_rsa
```

## 第 5 步：检查部署的 DM 集群情况

例如，执行如下命令检查 `dm-test` 集群情况：

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

预期输出包括 `dm-test` 集群中实例 ID、角色、主机、监听端口和状态（由于还未启动，所以状态为 Down/inactive）、目录信息。

## 第 6 步：启动集群

{{< copyable "shell-regular" >}}

```shell
tiup dm start dm-test
```

预期结果输出 ```Started cluster `dm-test` successfully``` 表示启动成功。

## 第 7 步：验证集群运行状态

通过以下 TiUP 命令检查集群状态：

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

在输出结果中，如果 Status 状态信息为 `Up`，说明集群状态正常。
