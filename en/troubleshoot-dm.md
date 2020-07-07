---
title: Data Migration Troubleshooting
summary: Learn how to diagnose and resolve issues when you use Data Migration.
category: reference
aliases: ['/docs/tidb-data-migration/stable/troubleshoot-dm/','/docs/tidb-data-migration/v1.0/troubleshoot-dm/','/docs/dev/how-to/troubleshoot/data-migration/','/docs/dev/reference/tools/data-migration/troubleshoot/dm/','/docs/v3.1/reference/tools/data-migration/troubleshoot/dm/','/docs/v3.0/reference/tools/data-migration/troubleshoot/dm/','/docs/v2.1/reference/tools/data-migration/troubleshoot/dm/']
---

# Data Migration Troubleshooting

This document describes how to troubleshoot TiDB Data Migration to find the problem.

If you encounter errors while running Data Migration, take the following steps to troubleshoot:

1. Execute the `query-status` command to view the task running status and related error output.

2. Check the log content related to the error you encountered. The log files are on the DM-master and DM-worker deployment nodes. To get key information about errors, see [common errors](error-system.md). Then check [error handling](error-handling.md) to find the corresponding solution.

3. If the error you encountered is not involved yet, and you cannot solve the problem yourself by checking the log or monitoring metrics, you can contact the corresponding sales support staff.

4. After the error is solved, restart the task using dmctl.

    ```bash
    resume-task ${task name}
    ```

However, you need to reset the data replication task in some cases. For details about when to reset and how to reset, see [Reset the data replication task](faq.md#how-to-reset-the-data-replication-task).
