---
title: 创建数据迁移任务
---

在创建数据迁移任务之前，你需要先完成
1. [使用 TiUP 部署 DM 集群](deploy-a-dm-cluster-using-tiup.md)
2. [创建数据源对象](quick-start-create-source.md)

# 创建数据迁移任务

本文介绍了在不同业务需求场景下如何配置数据迁移任务。 你可以通过场景的介绍，选择适合的教程来创建对应的数据迁移任务。

除了业务需求场景导向的创建数据迁移任务教程之外：

- 如果你需要完整的数据迁移任务配置示例，请参考 [DM 任务完整配置文件介绍](task-configuration-file-full.md)
- 如果你需要了解如何一步一步配置数据迁移任务，请参考 [数据迁移任务配置向导](task-configuration-guide.md)

## 多数据源汇总迁移到 TiDB
如果你需要将多个数据源的数据汇总迁移到 TiDB，此外还需要为多个数据源中冲突的表名进行重命名和屏蔽掉某些表的某些 DDL/DML 操作，那么你可以参考 [多数据源汇总迁移到 TiDB](usage-scenario-simple-migration.md)。

## 分库分表合并迁移到 TiDB
如果你需要将使用分表方案的业务合并迁移到 TiDB，那么你可以参考 [分表合并迁移到 TiDB](usage-scenario-shard-merge.md)。

## 只迁移数据源增量数据到 TiDB

如果你使用其他工具进行了全量数据迁移，例如使用 lightning 进行了全量数据迁移，然后使用 DM 只进行增量数据迁移，那么该场景的数据迁移任务配置可以参考 [只迁移数据源增量数据到 TiDB](usage-scenario-incremental-migration.md)。

## 只迁移数据源增量数据到 TiDB, 且TiDB 目标表比数据源表多列

如果你需要在 TiDB 定制创建表结构，TiDB 的表结构包含数据源对应表的所有列，且比数据源的表结构有更多的列，那么该场景的数据迁移任务配置需要参考 [TiDB 表结构存在更多列场景的数据迁移](usage-scenario-downstream-more-columns.md)。
