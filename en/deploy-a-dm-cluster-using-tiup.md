---
title: Deploy a DM Cluster Using TiUP
summary: Learn how to deploy TiDB Data Migration using TiUP DM.
aliases: ['/docs/tidb-data-migration/dev/deploy-a-dm-cluster-using-ansible/','/docs/tools/dm/deployment/','/tidb-data-migration/dev/deploy-a-dm-cluster-using-ansible']
---

# Deploy a DM Cluster Using TiUP

[TiUP](https://github.com/pingcap/tiup) is a cluster operation and maintenance tool introduced in TiDB 4.0. TiUP provides [TiUP DM](maintain-dm-using-tiup.md), a cluster management component written in Golang. By using TiUP DM, you can easily perform daily TiDB Data Migration (DM) operations, including deploying, starting, stopping, destroying, scaling, and upgrading a DM cluster, and manage DM cluster parameters.

TiUP supports deploying DM v2.0 or later DM versions. This document introduces how to deploy DM clusters of different topologies.

> **Note:**
>
> If your target machine's operating system supports SELinux, make sure that SELinux is **disabled**.

## Prerequisites

- [Hardware and software requirements](https://docs.pingcap.com/tidb-data-migration/stable/hardware-and-software-requirements)

## Step 1: Install TiUP on the control machine

Log in to the control machine using a regular user account (take the `tidb` user as an example). All the following TiUP installation and cluster management operations can be performed by the `tidb` user.

1. Install TiUP by executing the following command:

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

2. Set the TiUP environment variables:

    Redeclare the global environment variables:

    {{< copyable "shell-regular" >}}

    ```shell
    source .bash_profile
    ```

    Confirm whether TiUP is installed:

    {{< copyable "shell-regular" >}}

    ```shell
    which tiup
    ```

3. Install the TiUP DM component:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup install dm
    ```

4. If TiUP is already installed, update the TiUP DM component to the latest version:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update --self && tiup update dm
    ```

    Expected output includes `Update successfully!`.

## Step 2: Edit the initialization configuration file

According to the intended cluster topology, you need to manually create and edit the cluster initialization configuration file.

You need to create a YAML configuration file (named `topology.yaml` for example) according to the [configuration file template](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml). For other scenarios, edit the configuration accordingly.

You can use the command `tiup dm template > topology.yaml` to generate a configuration file template quickly.

The configuration of deploying three DM-masters, three DM-workers, and one monitoring component instance is as follows:

```yaml
# Global variables apply to all ways of deployment. If one specific value is missing, the corresponding global variable serves as the default value.
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/dm-deploy"
  data_dir: "/dm-data"

server_configs:
  master:
    log-level: info
    # rpc-timeout: "30s"
    # rpc-rate-limit: 10.0
    # rpc-rate-burst: 40
  worker:
    log-level: info

master_servers:
  - host: 10.0.1.11
    name: master1
    ssh_port: 22
    port: 8261
    # peer_port: 8291
    # deploy_dir: "/dm-deploy/dm-master-8261"
    # data_dir: "/dm-data/dm-master-8261"
    # log_dir: "/dm-deploy/dm-master-8261/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.master` values.
    config:
      log-level: info
      # rpc-timeout: "30s"
      # rpc-rate-limit: 10.0
      # rpc-rate-burst: 40
  - host: 10.0.1.18
    name: master2
    ssh_port: 22
    port: 8261
  - host: 10.0.1.19
    name: master3
    ssh_port: 22
    port: 8261

worker_servers:
  - host: 10.0.1.12
    ssh_port: 22
    port: 8262
    # deploy_dir: "/dm-deploy/dm-worker-8262"
    # log_dir: "/dm-deploy/dm-worker-8262/log"
    # numa_node: "0,1"
    # # Config is used to overwrite the `server_configs.dm-worker` values
    config:
      log-level: info
  - host: 10.0.1.19
    ssh_port: 22
    port: 8262

monitoring_servers:
  - host: 10.0.1.13
    ssh_port: 22
    port: 9090
    # deploy_dir: "/tidb-deploy/prometheus-8249"
    # data_dir: "/tidb-data/prometheus-8249"
    # log_dir: "/tidb-deploy/prometheus-8249/log"

grafana_servers:
  - host: 10.0.1.14
    port: 3000
    # deploy_dir: /tidb-deploy/grafana-3000

alertmanager_servers:
  - host: 10.0.1.15
    ssh_port: 22
    web_port: 9093
    # cluster_port: 9094
    # deploy_dir: "/tidb-deploy/alertmanager-9093"
    # data_dir: "/tidb-data/alertmanager-9093"
    # log_dir: "/tidb-deploy/alertmanager-9093/log"

```

> **Note:**
>
> - If you do not need to ensure high availability of the DM cluster, deploy only one DM-master node, and the number of deployed DM-worker nodes must be no less than the number of upstream MySQL/MariaDB instances to be migrated. To ensure high availability of the DM cluster, it is recommended to deploy three DM-master nodes, and the number of deployed DM-worker nodes must be greater than the number of upstream MySQL/MariaDB instances to be migrated (for example, the number of DM-worker nodes is two more than the number of upstream instances).
> 
> - It is not recommended to run too many DM-workers on one host. Each DM-worker should be allocated at least 2 core CPU and 4 GiB memory.
>
> - The worker will be bound with an upstream database, when DM performs a full replication task, the worker exporting the full amount of data locally first, and then importing it into the downstream database. Therefore, the worker's host needs sufficient storage space (The storage path will be specified later when creating the task)
> 
> - For parameters that should be globally effective, configure these parameters of corresponding components in the `server_configs` section of the configuration file. For parameters that should be effective on a specific node, configure these parameters in `config` of this node.
>
> - Use `.` to indicate the subcategory of the configuration, such as `log.slow-threshold`. For more formats, see [TiUP configuration template](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml).
>
> - For more parameter description, see [master `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/master/dm-master.toml) and [worker `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/worker/dm-worker.toml).
>
> - Make sure that the ports among the following components are interconnected:
>     - The `peer_port` (`8291` by default) among the DM-master nodes are interconnected.
>     - Each DM-master node can connect to the `port` of all DM-worker nodes (`8262` by default).
>     - Each DM-worker node can connect to the `port` of all DM-master nodes (`8261` by default).
>     - The TiUP nodes can connect to the `port` of all DM-master nodes (`8261` by default).
>     - The TiUP nodes can connect to the `port` of all DM-worker nodes (`8262` by default).

## Step 3: Execute the deployment command

> **Note:**
>
> You can use secret keys or interactive passwords for security authentication when you deploy TiDB using TiUP:
>
> - If you use secret keys, you can specify the path of the keys through `-i` or `--identity_file`;
> - If you use passwords, add the `-p` flag to enter the password interaction window;
> - If password-free login to the target machine has been configured, no authentication is required.

{{< copyable "shell-regular" >}}

```shell
tiup dm deploy ${name} ${version} ./topology.yaml -u ${ssh_user} [-p] [-i /home/root/.ssh/gcp_rsa]
```

The parameters used in this step is as follows.

|Parameter|Description|
|-|-|
|`${name}` | The name of dm cluster, eg: dm-test|
|`${version}` | The version of the DM cluster. You can see other supported versions by running `tiup list dm-master`.
|`./topology.yaml`| Path of topology config file.|
|`-u` or `--user`|Log in to the target machine through the root key to complete the cluster deployment, or you can use other users with ssh and sudo privileges to complete the deployment.|
|`-p` or `--password`|Password of target hosts. If specified, password authentication will be used|
|`-i` or `--identity_file`|The path of the SSH identity file. If specified, public key authentication will be used. (default "/root/.ssh/id_rsa")|

At the end of the output log, you will see ```Deployed cluster `dm-test` successfully```. This indicates that the deployment is successful.

## Step 4: Check the clusters managed by TiUP

{{< copyable "shell-regular" >}}

```shell
tiup dm list
```

TiUP supports managing multiple DM clusters. The command above outputs information of all the clusters currently managed by TiUP, including the name, deployment user, version, and secret key information:

```log
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
dm-test  tidb  v2.0.3  /root/.tiup/storage/dm/clusters/dm-test  /root/.tiup/storage/dm/clusters/dm-test/ssh/id_rsa
```

## Step 5: Check the status of the deployed DM cluster

To check the status of the `dm-test` cluster, execute the following command:

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

Expected output includes the instance ID, role, host, listening port, and status (because the cluster is not started yet, so the status is `Down`/`inactive`), and directory information.

## Step 6: Start the TiDB cluster

{{< copyable "shell-regular" >}}

```shell
tiup dm start dm-test
```

If the output log includes ```Started cluster `dm-test` successfully```, the start is successful.

## Step 7: Verify the running status of the TiDB cluster

Check the DM cluster status using TiUP:

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

If the `Status` is `Up` in the output, the cluster status is normal.

## Step 8: Get the cluster controller - dmctl

dmctl is a command-line tool used to control DM clusters. You are recommended to [use dmctl via TiUP](maintain-dm-using-tiup.md#dmctl).

dmctl supports both the command mode and the interactive mode. For details, see [Maintain DM Clusters Using dmctl](dmctl-introduction.md#maintain-dm- clusters-using-dmctl).
