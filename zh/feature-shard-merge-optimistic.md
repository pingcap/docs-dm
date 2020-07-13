---
title: 乐观模式下分库分表合并同步
category: reference
---

# 乐观模式下分库分表合并同步

本文介绍了 DM 提供的乐观模式下分库分表的合并同步功能，此功能可用于将上游 MySQL/MariaDB 实例中结构相同/不同的表同步到下游 TiDB 的同一个表中。

> **注意：**
>
> 在不深入了解乐观模式的原理和使用限制的情况下不建议使用该模式，否则可能造成同步中断甚至数据不一致的严重后果。

## 背景

DM 支持在线上执行分库分表的 DDL 语句（通称 Sharding DDL），默认使用“悲观模式”，即当上游一个分表执行某一 DDL 后，这个分表的同步会暂停，等待其他所有分表都执行了同样的 DDL 才在下游执行该 DDL 并继续数据同步。
这种“悲观协调”模式的优点是可以保证同步到下游的数据不会出错，问题是暂停同步进行不利于灰度。有些用户可能会花较长时间在单一分表执行 DDL，验证一定时间后后才会更改其他分表的结构。在悲观同步的设定下，这些 DDL 会阻塞同步，binlog 事件会大量积压。

因此，需要提供一种新的“乐观协调”模式，在一个分片上执行的 DDL，自动修改成兼容其他分片的语句后，立即同步到下游，不会阻挡任何分片 DML 的同步。

## 介绍

在“乐观协调”模式下， DM worker 接收到来自上游的 DDL 后，会把更新后的表结构转送给 DM master。DM worker 会各自追踪各分片当前的表结构，DM master 合并成可兼容来自每个分表 DML 的合成结构，然后把与此对应的 DDL 同步到下游；对于 DML 会直接同步到下游。

![optimistic-ddl-flow](/media/optimistic-ddl-flow.png)

### 例子

例如上游 MySQL 有三个分表，使用 DM 同步到下游 TiDB：

![optimistic-ddl-example-1](/media/optimistic-ddl-example-1.png)

在上游增加一列 `Level`：

```SQL
alter table `tbl00` add column `Level` int;
```

![optimistic-ddl-example-2](/media/optimistic-ddl-example-2.png)

此时下游 TiDB 要准备接受来自 tbl00 有 Level 的 DML、以及来自 tbl01 和 tbl02 没有 Level 的 DML。

![optimistic-ddl-example-3](/media/optimistic-ddl-example-3.png)

这时候各种 DML 无需修改都可以同步到下游。

```SQL
update `tbl00` set `Level` = 9 where `ID` = 1;
insert into `tbl02` (`ID`, `Name`) values (27, 'Tony');
```

![optimistic-ddl-example-4](/media/optimistic-ddl-example-4.png)

在 tbl01 同样增加一列 Level。

```SQL
alter table `tbl01` add column `Level` int;
```

![optimistic-ddl-example-5](/media/optimistic-ddl-example-5.png)

此时下游已经有相同的 Level 列了，所以 DM master 比较之后不做任何动作。

在 tbl01 刪除一列 Name。

```SQL
alter table `tbl01` drop column `Name`;
```

![optimistic-ddl-example-6](/media/optimistic-ddl-example-6.png)

此时下游仍需要接收来自 tbl00 和 tbl02 含 Name 的 DMLs，因此不会立刻删除该列。

同样，各种 DML 仍可直接同步到下游。

```SQL
insert into `tbl01` (`ID`, `Level`) values (15, 7);
update `tbl00` set `Level` = 5 where `ID` = 5;
```

![optimistic-ddl-example-7](/media/optimistic-ddl-example-7.png)

在 tbl02 增加一列 Level。

```SQL
alter table `tbl02` add column `Level` int;
```

![optimistic-ddl-example-8](/media/optimistic-ddl-example-8.png)

此时所有分片都已有 Level 列。

在 tbl00 和 tbl02 各刪除一列 Name。

```SQL
alter table `tbl00` drop column `Name`;
alter table `tbl02` drop column `Name`;
```

![optimistic-ddl-example-9](/media/optimistic-ddl-example-9.png)

到此步 Name 列也从所有分表消失了，所以可以安全从下游移除。

```SQL
alter table `tbl` drop column `Name`;
```

![optimistic-ddl-example-10](/media/optimistic-ddl-example-10.png)

### 风险 

使用乐观同步时，由于 DDL 会即时同步到下游，若使用不当，可能导致上下游数据不一致，并使整体结构重整期间的 DML 完全无效。

例如以下三个分片合并同步到 TiDB：

![optimistic-ddl-fail-example-1](/media/optimistic-ddl-fail-example-1.png)

在 tbl01 新增一列 Age，默认值定为 0。

```SQL
alter table `tbl01` add column `Age` int default 0;
alter table `tbl` add column `Age` int default 0;
```

![optimistic-ddl-fail-example-2](/media/optimistic-ddl-fail-example-2.png)

 在 tbl00 新增一列 Age，但默认值定为 -1。

```SQL 
alter table `tbl00` add column `Age` int default -1;
```

![optimistic-ddl-fail-example-3](/media/optimistic-ddl-fail-example-3.png)

此时所有来自 tbl00 的 Age 都不一致了。这是由于 default 0 和 default -1 互不兼容。虽然 DM 遇到这种情况会报错，但上下游不一致的问题就需要手动去解决。

### 使用限制

在乐观模式下，DM 可以允许上游分表有不同的表结构，也允许执行不同的 DDL 或者按不同的顺序执行相同的 DDL，但是仍然存在一定的限制。



## 使用

## 配置方法

## 错误处理

