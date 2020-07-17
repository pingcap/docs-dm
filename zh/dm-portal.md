---
title: DM Portal 简介
---

# DM Portal 简介

当前版本的 DM 提供了丰富多样的功能特性，包括 [Table routing](key-features.md#table-routing)，[Black & white table lists](key-features.md#black--white-table-lists)，[Binlog event filter](key-features.md#binlog-event-filter) 等。但这些功能特性同时也增加了用户使用 DM 的复杂度，尤其在进行 [DM 任务配置](task-configuration-file.md)的过程中。

针对这个问题，DM 提供了一个精简的网页程序 DM Portal，用于帮助用户以可视化的方式来配置所需的同步任务，并且生成可以让 DM 直接执行的 `task.yaml` 文件。

## 功能描述

本节简要介绍 DM Portal 的各项功能。

### 同步模式配置

支持 DM 的三种同步模式：

- 全量同步
- 增量同步
- All（全量+增量）

### 实例信息配置

支持配置库表同步路由方式，并支持 DM 中分库分表合并的配置方式。

### binlog 过滤配置

支持对数据库、数据表配置 binlog event filter 过滤规则。

### 配置文件生成

支持配置文件的创建，即能够将配置文件下载到本地并同时在 DM Portal 服务器的 `/tmp/` 目录下自动创建配置文件。

### 使用限制

目前，DM Portal 提供的可视化页面能够覆盖绝大部分的 DM 配置场景，但也有一定的使用限制：

- 不支持 [Binlog event filter](key-features.md#binlog-event-filter) 的 SQL pattern 方式
- 编辑功能不支持解析用户之前写的 `task.yaml` 文件，只支持编辑由页面生成的 `task.yaml` 文件
- 编辑功能不支持修改实例配置信息，如果用户需要调整实例配置，需要重新生成 `task.yaml` 文件
- 页面的上游实例配置仅用于获取上游库表结构，DM-worker 里依旧需要配置对应的上游实例信息

## 部署使用

本节介绍两种部署方式：Binary 部署和 DM-Ansible 部署。

### Binary 部署

DM Portal 的 binary 可以在对应版本的 DM 安装包中找到，通过 `./dm-portal` 的命令即可直接启动。

* 如果在本地启动，浏览器访问 `127.0.0.1:8280` 即可使用。
* 如果在服务器上启动，需要为服务器配置访问代理。

### DM Ansible 部署

可以使用 DM-Ansible 部署 DM Portal，具体部署方法参照[使用 DM Ansible 部署 DM 集群](deploy-a-dm-cluster-using-ansible.md)。

## 使用说明

本节介绍如何使用 DM Portal 各个功能。

### 1. 新建规则

#### 功能描述

新建一个 `task.yaml` 文件。

#### 操作步骤

登录 DM Portal 页面，点击**新建任务规则**。

### 2. 基础信息配置

#### 功能描述

用于填写任务名称，以及选择任务类型。

#### 前置条件

已选择**新建同步规则**。

#### 操作步骤

1. 填写任务名称。
2. 选择任务类型。

![DM Portal BasicConfig](/media/zh/dm-portal-basicconfig-zh.png)

### 3. 实例信息配置

#### 功能描述

用于配置上下游实例信息，包括 Host、Port、Username、Password。

#### 前置条件

已填写任务名称和选择任务类型。

#### 注意事项

如果任务类型选择**增量**或者 **All**，在配置上游实例信息时候，还需要配置 binlog-file 和 binlog-pos。

#### 操作步骤

1. 填写上游实例信息。
2. 填写下游实例信息。
3. 点击**下一步**。

![DM Portal InstanceConfig](/media/zh/dm-portal-instanceconfig-zh.png)

### 4. binlog 过滤配置

#### 功能描述

用于配置上游的 binlog 过滤规则，可以选择需要过滤的 DDL/DML。在数据库上配置的过滤规则，之后会自动被数据库下的数据表继承。

#### 前置条件

已经配置好上下游实例信息并且连接验证没问题。

#### 注意事项

* binlog 过滤配置只能在上游实例处进行修改编辑，一旦数据库或者数据表被移动到下游实例后，就不可以进行修改编辑。
* 在数据库上配置的 binlog 过滤规则会自动被其下的数据表继承。

#### 操作步骤

1. 点击需要配置的数据库或者数据表。

    ![DM Portal InstanceShow](/media/zh/dm-portal-instanceshow-zh.png)

2. 点击编辑按钮，选择需要过滤的 binlog 类型。

    ![DM Portal BinlogFilter 1](/media/zh/dm-portal-binlogfilter-1-zh.png)

    ![DM Portal BinlogFilter 2](/media/zh/dm-portal-binlogfilter-2-zh.png)

### 5. 库表路由配置

#### 功能描述

可以选择需要同步的数据库和数据表，并且进行修改名称、合并库、合并表等操作。可以对上一步操作进行撤销，可以对库表路由配置进行全部重置。在完成任务配置后，DM Portal 可以生成对应的 `task.yaml` 文件。

#### 前置条件

* 已经配置好需要的 binlog 过滤规则。

#### 注意事项

* 在合并库表操作的时候，不允许批量操作，只能逐一拖动。
* 在合表库表操作的时候，只能对数据表进行拖动操作，不能对数据库进行拖动操作。

#### 操作步骤

1. 在**上游实例**处，选择需要同步的数据库和数据表。

    ![DM Portal TableRoute 1](/media/zh/dm-portal-tableroute-1-zh.png)

2. 点击移动按钮，将需要同步的库表移动至**下游实例**处。

    ![DM Portal TableRoute 2](/media/zh/dm-portal-tableroute-2-zh.png)

3. 点击右键按钮，可以对库表进行改名操作。

    ![DM Portal ChangeTableName](/media/zh/dm-portal-changetablename-zh.png)

4. 选中需要操作的数据表，拖动至别的数据表图标上，可以对两个表进行合并。

    ![DM Portal MergeTable 1](/media/zh/dm-portal-mergetable-1-zh.png)

    ![DM Portal MergeTable 2](/media/zh/dm-portal-mergetable-2-zh.png)

    拖动到数据库图标上，可以将数据表移动至该库下。

    ![DM Portal MoveToDB 1](/media/zh/dm-portal-movetodb-1-zh.png)

    ![DM Portal MoveToDB 2](/media/zh/dm-portal-movetodb-2-zh.png)

    拖动到 target-instance 图标上，可以移动到一个新的数据库下。

    ![DM Portal MoveToNewDB 1](/media/zh/dm-portal-movetonewdb-1-zh.png)

    ![DM Portal MoveToNewDB 2](/media/zh/dm-portal-movetonewdb-2-zh.png)

5. 点击**完成**，自动下载 `task.yaml` 到本地，并且在 DM Portal 服务器上的 `/tmp/` 目录下自动创建一份 `task.yaml` 配置文件。

    ![DM Portal GenerateConfig](/media/zh/dm-portal-generateconfig-zh.png)

#### 其他操作

撤销本次操作：

![DM Portal Revert](/media/zh/dm-portal-revert-zh.png)

清空下游实例：

![DM Portal Reset](/media/zh/dm-portal-reset-zh.png)
