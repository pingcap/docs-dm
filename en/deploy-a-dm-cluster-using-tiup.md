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

You need to create a YAML configuration file (named `topology.yaml` for example) according to the [configuration file template](https://github.com/pingcap/tiup/blob/master/examples/dm/topology.example.yaml). For other scenarios, edit the configuration accordingly.

The configuration of deploying three DM-masters, three DM-workers, and one monitoring component instance is as follows:

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

> **Note:**
>
> - If you do not need to ensure high availability of the DM cluster, deploy only one DM-master node, and the number of deployed DM-worker nodes must be no less than the number of upstream MySQL/MariaDB instances to be migrated.
>
> - To ensure high availability of the DM cluster, it is recommended to deploy three DM-master nodes, and the number of deployed DM-worker nodes must be greater than the number of upstream MySQL/MariaDB instances to be migrated (for example, the number of DM-worker nodes is two more than the number of upstream instances).
>
> - For parameters that should be globally effective, configure these parameters of corresponding components in the `server_configs` section of the configuration file.
>
> - For parameters that should be effective on a specific node, configure these parameters in `config` of this node.
>
> - Use `.` to indicate the subcategory of the configuration, such as `log.slow-threshold`. For more formats, see [TiUP configuration template](https://github.com/pingcap/tiup/blob/master/examples/dm/topology.example.yaml).
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
tiup dm deploy dm-test ${version} ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```

In the above command:

- The name of the deployed DM cluster is `dm-test`.
- The version of the DM cluster is `${version}`. You can see other supported versions by running `tiup list dm-master`.
- The initialization configuration file is `topology.yaml`.
- `--user root`: Log in to the target machine through the `root` key to complete the cluster deployment, or you can use other users with `ssh` and `sudo` privileges to complete the deployment.
- `[-i]` and `[-p]`: optional. If you have configured login to the target machine without password, these parameters are not required. If not, choose one of the two parameters. `[-i]` is the private key of the `root` user (or other users specified by `--user`) that has access to the target machine. `[-p]` is used to input the user password interactively.
- TiUP DM uses the embedded SSH client. If you want to use the SSH client native to the control machine system, edit the configuration according to [using the system's native SSH client to connect to the cluster](maintain-dm-using-tiup.md#use-the-systems-native-ssh-client-to-connect-to-cluster).

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
dm-test  tidb  v2.0.0  /root/.tiup/storage/dm/clusters/dm-test  /root/.tiup/storage/dm/clusters/dm-test/ssh/id_rsa
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
