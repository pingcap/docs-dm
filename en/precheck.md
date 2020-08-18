---
title: Precheck the upstream MySQL instance configuration
summary: Use the precheck feature provided by DM to detect errors in the upstream MySQL instance configuration.
aliases: ['/docs/tidb-data-migration/stable/precheck/','/docs/tidb-data-migration/v1.0/precheck/','/docs/dev/reference/tools/data-migration/precheck/','/docs/v3.1/reference/tools/data-migration/precheck/','/docs/v3.0/reference/tools/data-migration/precheck/','/docs/v2.1/reference/tools/data-migration/precheck/']
---

# Precheck the upstream MySQL instance configuration

This document introduces the precheck feature provided by DM. This feature is used to detect possible errors in the upstream MySQL instance configuration when the data replication task is started.

## Command

`check-task` allows you to precheck whether the upstream MySQL instance configuration satisfies the DM requirements.

## Checking items

Upstream and downstream database users must have the corresponding read and write privileges. DM checks the following privileges and configuration automatically while the data replication task is started:

+ Database version

    - 5.5 < MySQL version < 8.0
    - MariaDB version >= 10.1.2

+ MySQL binlog configuration

    - Whether the binlog is enabled (DM requires that the binlog must be enabled)
    - Whether `binlog_format=ROW` (DM only supports replication of the binlog in the ROW format)
    - Whether `binlog_row_image=FULL` (DM only supports `binlog_row_image=FULL`)

+ The privileges of the upstream MySQL instance users

    MySQL users in DM configuration need to have the following privileges at least:

    - REPLICATION SLAVE
    - REPLICATION CLIENT
    - RELOAD
    - SELECT

+ The compatibility of the upstream MySQL table schema

    TiDB differs from MySQL in compatibility in the following aspects:

    - TiDB does not support the foreign key.
    - [Character set compatibility differs](https://pingcap.com/docs/stable/reference/sql/character-set/).

    DM will also check whether the primary key or unique key restriction exists in all upstream tables. This check is introduced in v1.0.7.

+ The consistency of the sharded tables in the multiple upstream MySQL instances

    + The schema consistency of all sharded tables

        - Column size
        - Column name
        - Column position
        - Column type
        - Primary key
        - Unique index

    + The conflict of the auto increment primary keys in the sharded tables

        - The check fails in the following two conditions:

            - The auto increment primary key exists in the sharded tables and its column type *is not* bigint.
            - The auto increment primary key exists in the sharded tables and its column type *is* bigint, but column mapping *is not* configured.

        - The check succeeds in other conditions except the two above.
