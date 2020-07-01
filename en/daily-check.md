---
title: TiDB Data Migration daily inspection
summary: Understand the daily inspection of DM.
category: reference
---

# TiDB Data Migration Daily inspection

This document summarizes the daily inspection methods of the TiDB Data Migration (DM):

+ Method 1: Execute the `query-status` command to view the task running status and related error output. See [Query Status](query-status.md) for details.

+ Method 2: If Prometheus and Grafana are correctly deployed when using DM-Ansible to deploy the DM cluster, then you can veiw DM's dashboard in Grafana. For example, Grafana's address is `172.16.10.71`, you can open <http://172.16.10.71:3000> in your browser to enter the Grafana, and then choose DM's Dashboard to view DM related monitoring items. Refer to [Data Migration Monitoring Metrics](monitor-a-dm-cluster.md) for specific monitoring items.

+ Method 3: Check the DM running status and related errors through the log file.

    - DM-master's log directory: set by the DM-master's parameter `--log-file`. If DM-Ansible is used to deploy DM, the log directory is located in `{ansible deploy}/log/dm-master.log` of DM-master node.
    - DM-worker's log directory: set by DM-worker's parameter `--log-file`. If DM-Ansible is used to deploy DM, the log directory is located in `{ansible deploy}/log/dm-worker.log` of the DM-worker node.
