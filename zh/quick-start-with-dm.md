---
title: TiDB Data Migration 快速上手指南
summary: 了解如何快速上手部署试用 TiDB Data Migration 工具。
aliases: ['/docs-cn/tidb-data-migration/dev/quick-start-with-dm/','/docs-cn/tidb-data-migration/dev/get-started/']
---

# TiDB Data Migration 快速上手指南

本文将介绍如何快速上手体验数据迁移工具 [TiDB Data Migration](https://github.com/pingcap/dm) (DM)。体验方式为使用 binary 包部署 DM。

## 使用样例

在本地部署 DM-master 与 DM-worker 实例。各个节点的信息如下：

| 实例        | 服务器地址   | 端口使用 |
| :---------- | :----------- | :----------- |
| DM-master1 | 127.0.0.1 | 8261, 8291 |
| DM-master2 | 127.0.0.1 | 8361, 8292 |
| DM-master3 | 127.0.0.1 | 8461, 8293 |
| DM-worker1 | 127.0.0.1 | 8262 |
| DM-worker2 | 127.0.0.1 | 8263 |
| DM-worker3 | 127.0.0.1 | 8264 |

## 准备工作

部署 DM 前需要下载 binary，搭建好上下游数据库，并准备好数据。

### 准备 DM binary 包

首先需要下载 DM 2.0 的 binary 或者手动编译。

#### 第一种方式：下载最新 DM binary 包

{{< copyable "shell-regular" >}}

```bash
wget http://download.pingcap.org/dm-nightly-linux-amd64.tar.gz
tar -xzvf dm-nightly-linux-amd64.tar.gz
cd dm-nightly-linux-amd64
```

#### 第二种方式：编译最新 DM binary 包

{{< copyable "shell-regular" >}}

```bash
git clone https://github.com/pingcap/dm.git
cd dm
make
```

#### 可选项：将下载/编译的 binary 加入环境变量 PATH 中，方便部署使用

{{< copyable "shell-regular" >}}

```bash
DM_PATH=`pwd` && export PATH=$PATH:$DM_PATH/bin
```

## 部署 DM-master

分别创建 master1、master2、master3 三个目录，在每个目录下创建 DM-master 的配置文件，配置文件如下：

`master1/dm-master1.toml`:

```toml
# DM-master1 Configuration.
name = "master1"
master-addr = ":8261"
advertise-addr = "127.0.0.1:8261"
peer-urls = "127.0.0.1:8291"
initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
```

`master2/dm-master2.toml`:

```toml
# DM-master2 Configuration.
name = "master2"
master-addr = ":8361"
advertise-addr = "127.0.0.1:8361"
peer-urls = "127.0.0.1:8292"
initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
```

`master3/dm-master3.toml`:

```toml
# DM-master3 Configuration.
name = "master3"
master-addr = ":8461"
advertise-addr = "127.0.0.1:8461"
peer-urls = "127.0.0.1:8293"
initial-cluster = "master1=http://127.0.0.1:8291,master2=http://127.0.0.1:8292,master3=http://127.0.0.1:8293"
```

分别进入 master1、master2、master3 三个目录，执行如下命令启动 DM-master：

{{< copyable "shell-regular" >}}

```bash
nohup dm-master --config=dm-master1.toml --log-file=dm-master1.log >> dm-master1.log 2>&1 &
nohup dm-master --config=dm-master2.toml --log-file=dm-master2.log >> dm-master2.log 2>&1 &
nohup dm-master --config=dm-master3.toml --log-file=dm-master3.log >> dm-master3.log 2>&1 &
```

## 部署 DM-worker

分别创建 worker1、worker2、worker3 三个目录，在每个目录下创建 DM-worker 的配置文件，配置文件如下：

`worker1/dm-worker1.toml`:

```toml
# DM-worker1 Configuration
name = "worker1"
worker-addr="0.0.0.0:8262"
advertise-addr="127.0.0.1:8262"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

`worker2/dm-worker2.toml`:

```toml
# DM-worker2 Configuration
name = "worker2"
worker-addr="0.0.0.0:8263"
advertise-addr="127.0.0.1:8263"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

`worker3/dm-worker3.toml`:

```toml
# DM-worker3 Configuration
name = "worke3"
worker-addr="0.0.0.0:8264"
advertise-addr="127.0.0.1:8264"
join = "127.0.0.1:8261,127.0.0.1:8361,127.0.0.1:8461"
```

分别进入 worker1、worker2、worker3 三个目录，执行如下命令启动 DM-worker：

{{< copyable "shell-regular" >}}

```bash
nohup dm-worker --config=dm-worker1.toml --log-file=dm-worker1.log >> dm-worker1.log 2>&1 &
nohup dm-worker --config=dm-worker2.toml --log-file=dm-worker2.log >> dm-worker2.log 2>&1 &
nohup dm-worker --config=dm-worker3.toml --log-file=dm-worker3.log >> dm-worker3.log 2>&1 &
```

## 检查 DM 集群部署是否正常

{{< copyable "shell-regular" >}}

```bash
dmctl --master-addr=127.0.0.1:8261 list-member
```

检查返回结果中是否有 leader 项，同时检查 master 与 worker 项是否包含了所有的 master 与 worker 拓扑。

一个正常 DM 集群的范例返回结果如下所示：

```bash
{
    "result": true,
    "msg": "",
    "members": [
        {
            "leader": {
                "msg": "",
                "name": "master1",
                "addr": "127.0.0.1:8261"
            }
        },
        {
            "master": {
                "msg": "",
                "masters": [
                    {
                        "name": "master1",
                        "memberID": "11007177379717700053",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8291"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8261"
                        ]
                    },
                    {
                        "name": "master2",
                        "memberID": "12007177379717800042",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8292"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8361"
                        ]
                    },
                    {
                        "name": "master3",
                        "memberID": "13007157379717700087",
                        "alive": true,
                        "peerURLs": [
                            "http://127.0.0.1:8293"
                        ],
                        "clientURLs": [
                            "http://0.0.0.0:8461"
                        ]
                    },
                ]
            }
        },
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "worker1",
                        "addr": "127.0.0.1:8262",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "worker2",
                        "addr": "127.0.0.1:8263",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "worker3",
                        "addr": "127.0.0.1:8264",
                        "stage": "free",
                        "source": ""
                    }
                ]
            }
        }
    ]
}
```
