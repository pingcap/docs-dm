---
title: Enable TLS for DM Connections
summary: Learn how to enable TLS for DM connections.
---

# Enable TLS for DM Connections

This document describes how to enable encrypted data transmission for DM connections, including connections between the DM-master, DM-worker, and dmctl components, and connections between DM and the upstream or downstream database.

## Enable encrypted data transmission between DM-master, DM-worker, and dmctl

This section introduces how to enable encrypted data transmission between DM-master, DM-worker, and dmctl.

### Configure and enable encrypted data transmission

1. Prepare certificates.

    It is recommended to prepare a server certificate for DM-master and DM-worker separately. Make sure that the two components can authenticate each other. You can choose to share one client certificate for dmctl.

    You can use tools like `openssl`, `easy-rsa` and `cfssl` to generate self-signed certificates.

    If you choose `openssl`, you can refer to [generating self-signed certificates](generate-self-signed-certificates.md).

2. Configure certificates.

    - DM-master

        Configure in the configuration file or command-line arguments:

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/cert.pem"
        ssl-key = "/path/to/key.pem"
        ```

    - DM-worker

        Configure in the configuration file or command-line arguments:

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/cert.pem"
        ssl-key = "/path/to/key.pem"
        ```

    - dmctl

        After enabling encrypted transmission in a DM cluster, if you need to connect to the cluster using dmctl, specify the client certificate. For example:

        {{< copyable "shell-regular" >}}

        ```bash
        ./dmctl -master-addr=127.0.0.1:8261 --ssl-ca /path/to/client-ca.pem --ssl-cert /path/to/client-cert.pem --ssl-key /path/to/client-key.pem
        ```

### Verify component caller's identity

The Common Name is used for caller verification. In general, the callee needs to verify the caller's identity, in addition to verifying the key, the certificates, and the CA provided by the caller. For example, DM-worker can only be accessed by DM-master, and other visitors are blocked even though they have legitimate certificates.

To verify component caller's identity, you need to mark the certificate user identity using `Common Name` when generating the certificate, and to check the caller's identity by configuring the `Common Name` list for the callee.

- DM-master

    Configure in the configuration file or command-line arguments:

    ```toml
    cert-allowed-cn = ["dm"]
    ```

- DM-worker

    Configure in the configuration file or command-line arguments:

    ```toml
    cert-allowed-cn = ["dm"]
    ```

### Reload certificates

To reload the certificates and the keys, DM-master, DM-worker, and dmctl reread the current certificates and the key files each time a new connection is created.

## Enable encrypted data transmission between DM components and the upstream or downstream database

This section introduces how to enable encrypted data transmission between DM components and the upstream or downstream database.

### Enable encrypted data transmission for upstream database

1. Configure the upstream database, enable the encryption support, and set the server certificate. For detailed operations, see [Using encrypted connections](https://dev.mysql.com/doc/refman/5.7/en/using-encrypted-connections.html).

2. Set the client certificate in the source configuration file:

    ```yaml
    from:
        security:
            ssl-ca: "/path/to/client-ca.pem"
            ssl-cert: "/path/to/client-cert.pem"
            ssl-key: "/path/to/client-key.pem"
    ```

### Enable encrypted data transmission for downstream TiDB

1. Configure the downstream TiDB to use encrypted connections. For detailed operatons,  refer to [Configure TiDB to use encrypted connections](https://docs.pingcap.com/tidb/stable/enable-tls-between-clients-and-servers#configure-tidb-to-use-encrypted-connections).

2. Set the client certificate in the task configuration file:

    ```yaml
    target-database:
        security:
            ssl-ca: "/path/to/client-ca.pem"
            ssl-cert: "/path/to/client-cert.pem"
            ssl-key: "/path/to/client-key.pem"
    ```
