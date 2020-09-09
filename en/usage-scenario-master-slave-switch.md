---
title: Switch DM-worker Connection between Upstream MySQL Instances
summary: Learn how to switch the DM-worker connection between upstream MySQL instances.
---

# Switch DM-worker Connection between Upstream MySQL Instances

When the upstream MySQL instance that DM-worker connects to needs downtime maintenance or when the instance crashes unexpectedly, you need to switch the DM-worker connection to another MySQL instance within the same migration group.

> **Note:**
>
> - You can switch the DM-worker connection to only an instance within the same primary-secondary migration cluster.
> - The MySQL instance to be newly connected to must have the binlog required by DM-worker.
> - DM-worker must operate in the GTID sets mode, which means you must specify `enable_gtid=true` when you deploy DM using DM-Ansible.
> - The connection switch only supports the following two scenarios. Strictly follow the procedures for each scenario. Otherwise, you might have to re-deploy the DM cluster according to the newly connected MySQL instance and perform the data migration task all over again.

For more details on GTID set, refer to [MySQL documentation](https://dev.mysql.com/doc/refman/5.7/en/replication-gtids-concepts.html#replication-gtids-concepts-gtid-sets).

## Switch DM-worker connection via virtual IP

When DM-worker connects the upstream MySQL instance via a virtual IP (VIP), switching the VIP connection to another MySQL instance means switching the MySQL instance connected to DM-worker, without the upstream connection address changed.

> **Note:**
>
> Make necessary changes to DM in this scenario. Otherwise, when you switch the VIP connection to another MySQL instance, DM might connect to the new and old MySQL instances at the same time in different connections. In this situation, the binlog replicated to DM is not consistent with other upstream status that DM receives, causing unpredictable anomalies and even data damage.

To switch one upstream MySQL instance (when DM-worker connects to it via a VIP) to another, perform the following steps:

1. Use the `query-status` command to get the GTID sets (`relayBinlogGtid`) corresponding to the binlog that relay log has replicated from the old MySQL instance. Mark the sets as `gtid-W`.
2. Use the `SELECT @@GLOBAL.gtid_purged;` command on the new MySQL instance to get the GTID sets corresponding to the purged binlogs. Mark the sets as `gtid-P`.
3. Use the `SELECT @@GLOBAL.gtid_executed;` command on the new MySQL instance to get the GTID sets corresponding to all successfully executed transactions. Mark the sets as `gtid-E`.
4. Make sure that the following conditions are met. Otherwise, you cannot switch the DM-work connection to the new MySQL instance:
    - `gtid-W` contains `gtid-P`. `gtid-P` can be empty.
    - `gtid-E` contains `gtid-W`.
5. Use `pause-relay` to pause relay.
6. Use `pause-task` to pause all running tasks of data migration.
7. Change the VIP for it to direct at the new MySQL instance.
8. Use `switch-relay-master` to tell relay to execute the primary-secondary switch.
9. Use `resume-relay` to make relay resume to read binlog from the new MySQL instance.
10. Use `resume-task` to resume the previous migration task.

## Change the address of the upstream MySQL instance that DM-worker connects to

To make DM-worker connect to a new MySQL instance in the upstream by modifying the DM-worker configuration, perform the following steps:

1. Use the `query-status` command to get the GTID sets (`relayBinlogGtid`) corresponding to the binlog that relay log has replicated from the old MySQL instance. Mark this sets as `gtid-W`.
2. Use the `SELECT @@GLOBAL.gtid_purged;` command on the new MySQL instance to get the GTID sets corresponding to the purged binlogs. Mark this sets as `gtid-P`.
3. Use the `SELECT @@GLOBAL.gtid_executed;` command on the new MySQL instance to get the GTID sets corresponding to all successfully executed transactions. Mark this sets as `gtid-E`.
4. Make sure that the following conditions are met. Otherwise, you cannot switch the DM-work connection to the new MySQL instance:
    - `gtid-W` contains `gtid-P`. `gtid-P` can be empty.
    - `gtid-E` contains `gtid-W`.
5. Use `stop-task` to stop all running tasks of data migration.
6. Update the DM-worker configuration in the `inventory.ini` file and use DM-Ansible to perform a rolling upgrade on DM-worker.
7. Use `start-task` to restart the migration task.
