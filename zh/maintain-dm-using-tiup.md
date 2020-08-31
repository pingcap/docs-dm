---
title: 使用 TiUP 运维 DM 集群
summary: 学习如何使用 TiUP 运维 DM 集群。
aliases: ['/docs-cn/tidb-data-migration/dev/cluster-operations/','/zh/tidb-data-migration/dev/cluster-operations']
---

# 使用 TiUP 运维 DM 集群

本文介绍如何使用 TiUP 的 DM 组件运维 DM 集群。使用 TiUP 部署 DM 的完整步骤可参考[使用 TiUP 部署 DM 集群](deploy-a-dm-cluster-using-tiup.md)。

TiUP DM 组件的帮助信息如下：

```bash
tiup dm --help
```

```
Deploy a DM cluster for production

Usage:
  tiup dm [flags]
  tiup dm [command]

Available Commands:
  deploy      Deploy a DM cluster for production
  start       Start a DM cluster
  stop        Stop a DM cluster
  restart     Restart a DM cluster
  list        List all clusters
  destroy     Destroy a specified DM cluster
  audit       Show audit log of cluster operation
  exec        Run shell command on host in the dm cluster
  edit-config Edit DM cluster config
  display     Display information of a DM cluster
  reload      Reload a DM cluster's config and restart if needed
  upgrade     Upgrade a specified DM cluster
  patch       Replace the remote package with a specified package and restart the service
  scale-out   Scale out a DM cluster
  scale-in    Scale in a DM cluster
  import      Import an exist DM 1.0 cluster from dm-ansible and re-deploy 2.0 version
  help        Help about any command

Flags:
  -h, --help               help for tiup-dm
      --native-ssh         Use the native SSH client installed on local system instead of the build-in one.
      --ssh-timeout int    Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
  -v, --version            version for tiup-dm
      --wait-timeout int   Timeout in seconds to wait for an operation to complete, ignored for operations that don't fit. (default 60)
  -y, --yes                Skip all confirmations and assumes 'yes'
```

## 查看集群列表

集群部署成功后，可以通过 `tiup dm list` 命令在集群列表中查看该集群：

{{< copyable "shell-root" >}}

```bash
tiup dm list
```

```
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
prod-cluster  tidb  v2.0.0  /root/.tiup/storage/dm/clusters/test  /root/.tiup/storage/dm/clusters/test/ssh/id_rsa
```

## 启动集群

集群部署成功后，可以执行以下命令启动该集群。如果忘记了部署的集群名字，可以使用 `tiup dm list` 命令查看。

{{< copyable "shell-regular" >}}

```shell
tiup start start prod-cluster
```

## 检查集群状态

如果想查看集群中每个组件的运行状态，逐一登录到各个机器上查看显然很低效。因此，TiUP 提供了 `tiup dm display` 命令，用法如下：

{{< copyable "shell-root" >}}

```bash
tiup dm display prod-cluster
```

```
dm Cluster: prod-cluster
dm Version: v2.0.0
ID                 Role          Host          Ports      OS/Arch       Status     Data Dir                           Deploy Dir
--                 ----          ----          -----      -------       ------     --------                           ----------
172.19.0.101:9093  alertmanager  172.19.0.101  9093/9094  linux/x86_64  Up         /home/tidb/data/alertmanager-9093  /home/tidb/deploy/alertmanager-9093
172.19.0.101:8261  dm-master     172.19.0.101  8261/8291  linux/x86_64  Healthy|L  /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.102:8261  dm-master     172.19.0.102  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.103:8261  dm-master     172.19.0.103  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.101:8262  dm-worker     172.19.0.101  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.102:8262  dm-worker     172.19.0.102  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.103:8262  dm-worker     172.19.0.103  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.101:3000  grafana       172.19.0.101  3000       linux/x86_64  Up         -                                  /home/tidb/deploy/grafana-3000
172.19.0.101:9090  prometheus    172.19.0.101  9090       linux/x86_64  Up         /home/tidb/data/prometheus-9090    /home/tidb/deploy/prometheus-9090
```

Status 列用 `Up` 或者 `Down` 表示该服务是否正常。对于 master 组件，同时可能会带有 `|L` 表示该 master 是 Leader, 对于 worker 组件，Free 表示当前 worker 没有与上游绑定。

## 缩容节点

缩容即下线服务，最终会将指定的节点从集群中移除，并删除遗留的相关数据文件。

缩容操作进行时，内部对 dm-master、dm-worker 组件的操作流程为：

1. 停止组件进程
2. 调用 DM master 删除 member 的 API
3. 清除节点的相关数据文件

缩容命令的基本用法：

```bash
tiup dm scale-in <cluster-name> -N <node-id>
```

它需要指定至少两个参数，一个是集群名字，另一个是节点 ID。节点 ID 可以参考上一节使用 `tiup dm display` 命令获取。

比如想缩容 172.16.5.140 上的 worker 节点，可以执行：

{{< copyable "shell-regular" >}}

```bash
tiup dm scale-in prod-cluster -N 172.16.5.140:8262
```

## 扩容节点

扩容的内部逻辑与部署类似，TiUP DM 组件会先保证节点的 SSH 连接，在目标节点上创建必要的目录，然后执行部署并且启动服务。

例如，在集群 `prod-cluster` 中扩容一个 worker 节点：

1. 新建 scale.yaml 文件，添加新增的 woker 节点信息：

    > **注意：**
    >
    > 需要新建一个拓扑文件，文件中只写入扩容节点的描述信息，不要包含已存在的节点。

    ```yaml
    ---

    worker_servers:
      - host: 172.16.5.140

    ```

2. 执行扩容操作。TiUP DM 根据 scale.yaml 文件中声明的端口、目录等信息在集群中添加相应的节点：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dm scale-out prod-cluster scale.yaml
    ```

    执行完成之后可以通过 `tiup dm display prod-cluster` 命令检查扩容后的集群状态。

## 滚动升级

滚动升级过程中尽量保证对前端业务透明、无感知，其中对不同节点有不同的操作。

### 升级操作

可使用 `tiup dm upgrade` 命令来升级集群。例如，把集群升级到 v2.0.1 的命令为：

{{< copyable "shell-regular" >}}

```bash
tiup dm upgrade prod-cluster v2.0.1
```

## 更新配置

如果想要动态更新组件的配置，TiUP DM 组件为每个集群保存了一份当前的配置，如果想要编辑这份配置，则执行 `tiup dm edit-config <cluster-name>` 命令。例如：

{{< copyable "shell-regular" >}}

```bash
tiup dm edit-config prod-cluster
```

然后 TiUP DM 组件会使用 vi 打开配置文件供编辑（如果你想要使用其他编辑器，请使用 `EDITOR` 环境变量自定义编辑器，例如 `export EDITOR=nano`），编辑完之后保存即可。此时的配置并没有应用到集群，如果想要让它生效，还需要执行：

{{< copyable "shell-regular" >}}

```bash
tiup dm reload prod-cluster
```

该操作会将配置发送到目标机器，滚动重启集群，使配置生效。

## 更新组件

常规的升级集群可以使用 `upgrade` 命令，但是在某些场景下（例如 Debug)，可能需要用一个临时的包替换正在运行的组件，此时可以用 `patch` 命令：

{{< copyable "shell-root" >}}

```bash
tiup dm patch --help
```

```
Replace the remote package with a specified package and restart the service

Usage:
  tiup dm patch <cluster-name> <package-path> [flags]

Flags:
  -h, --help                   help for patch
  -N, --node strings           Specify the nodes
      --overwrite              Use this package in the future scale-out operations
  -R, --role strings           Specify the role
      --transfer-timeout int   Timeout in seconds when transferring dm-master leaders (default 300)

Global Flags:
      --native-ssh         Use the native SSH client installed on local system instead of the build-in one.
      --ssh-timeout int    Timeout in seconds to connect host via SSH, ignored for operations that don't need an SSH connection. (default 5)
      --wait-timeout int   Timeout in seconds to wait for an operation to complete, ignored for operations that don't fit. (default 60)
  -y, --yes                Skip all confirmations and assumes 'yes'
```

例如，有一个 dm-master 的 hotfix 包放在 `/tmp/dm-master-hotfix.tar.gz`，如果此时想要替换集群上的所有 master，则可以执行：

{{< copyable "shell-regular" >}}

```bash
tiup dm patch prod-cluster /tmp/dm-master-hotfix.tar.gz -R dm-master
```

或者只替换其中一个 master：

{{< copyable "shell-regular" >}}

```bash
tiup dm patch prod-cluster /tmp/dm--hotfix.tar.gz -N 172.16.4.5:8261
```

## 导入 DM-Ansible 部署的 DM 1.0 集群并升级

> **注意：**
>
> - TiUP 不支持导入 1.0 集群中的 DM Portal 组件。
> - 导入前请先停止原集群。
> - 部分组件生成的部署目录会跟原集群不一样，具体可以使用 `display` 命令查看。
> - 导入前运行 `tiup update --self && tiup update dm` 确认升级 TiUP DM 组件到最新版本。

在 TiUP 之前，一般使用 DM Ansible 部署 DM 集群，`import` 命令用于根据 Ansible 部署的 1.0 集群生成 TiUP 对应的 `topology.yaml`, 并根据拓扑部署 2.0 的集群。

例如，导入一个 DM Ansible 集群：

{{< copyable "shell-regular" >}}

```bash
tiup dm import --dir=/path/to/tidb-ansible --cluster-version v2.0.0-rc
```

可以通过执行 `tiup list dm-master` 来查看 TiUP 支持的最新集群版本。

`import` 命令的工作流程如下：

- 根据 ansible 部署的集群生成一个 TiUP 部署使用的拓扑文件 [topology.yml](https://github.com/pingcap/tiup/blob/master/examples/topology.dm.example.yaml)。
- 确认部署后使用生成的拓扑文件部署 2.0 以上版本的集群。

部署成功后可以使用 `tiup dm start` 命令启动集群后进入 DM 内核升级流程。

## 查看操作日志

操作日志的查看可以借助 `audit` 命令，其用法如下：

```bash
Usage:
  tiup dm audit [audit-id] [flags]

Flags:
  -h, --help   help for audit
```

在不使用 `[audit-id]` 参数时，该命令会显示执行的命令列表，如下：

{{< copyable "shell-regular" >}}

```bash
tiup dm audit
```

```
ID      Time                  Command
--      ----                  -------
4D5kQY  2020-08-13T05:38:19Z  tiup dm display test
4D5kNv  2020-08-13T05:36:13Z  tiup dm list
4D5kNr  2020-08-13T05:36:10Z  tiup dm deploy -p prod-cluster v2.0.0 ./examples/dm/minimal.yaml
```

第一列为 audit-id，如果想看某个命令的执行日志，则传入这个 audit-id：

{{< copyable "shell-regular" >}}

```bash
tiup dm audit 4D5kQY
```

## 在集群节点机器上执行命令

`exec` 命令可以很方便地到集群的机器上执行命令，使用方式如下：

```bash
Usage:
  tiup dm exec <cluster-name> [flags]

Flags:
      --command string   the command run on cluster host (default "ls")
  -h, --help             help for exec
  -N, --node strings     Only exec on host with specified nodes
  -R, --role strings     Only exec on host with specified roles
      --sudo             use root permissions (default false)
```

例如，如果要到所有的 DM 节点上执行 `ls /tmp`，则可以执行：

{{< copyable "shell-regular" >}}

```bash
tiup dm exec prod-cluster --command='ls /tmp'
```

## 集群控制工具 (dmctl)

TiUP 集成了 DM 的控制工具 `dmctl`：

运行命令如下：

```bash
tiup dmctl [args]
```

指定 dmctl 版本：

```
tiup dmctl:v2.0.0 [args]
```

例如，以前添加 source 命令为 `dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml`，集成到 TiUP 中的命令为：

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml
```

## 使用中控机系统自带的 SSH 客户端连接集群

在以上所有操作中，涉及到对集群机器的操作都是通过 TiUP 内置的 SSH 客户端连接集群执行命令，但是在某些场景下，需要使用系统自带的 SSH 客户端来对集群执行操作，比如：

- 使用 SSH 插件来做认证
- 使用定制的 SSH 客户端

此时可以通过命令行参数 `--native-ssh` 启用系统自带命令行：

- 部署集群: `tiup dm deploy <cluster-name> <version> <topo> --native-ssh`
- 启动集群: `tiup dm start <cluster-name> --native-ssh`
- 升级集群: `tiup dm upgrade ... --native-ssh`

所有涉及集群操作的步骤都可以加上 `--native-ssh` 来使用系统自带的客户端。

也可以使用环境变量 `TIUP_NATIVE_SSH` 来指定是否使用本地 SSH 客户端，避免每个命令都需要添加 `--native-ssh` 参数：

```sh
export TIUP_NATIVE_SSH=true
# 或者
export TIUP_NATIVE_SSH=1
# 或者
export TIUP_NATIVE_SSH=enable
```

若环境变量和 `--native-ssh` 同时指定，则以 `--native-ssh` 为准。

> **注意：**
>
> 在部署集群的步骤中，若需要使用密码的方式连接 (-p)，或者密钥文件设置了 passphrase，则需要保证中控机上安装了 sshpass，否则连接时会报错。
