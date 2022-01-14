BlobStore部署文档
=================


编译构建
--------

.. code-block:: bash

   $ git clone https://github.com/chubaofs/blobstore.git
   $ cd blobstore
   $ source env.sh
   $ ./build.sh

如果构建成功，将在 ``bin`` 目录中生成以下可执行文件：

    1. clustermgr
    2. blobnode
    3. allocator
    4. mqproxy
    5. access
    6. scheduler
    7. tinker
    8. worker
    9. cli

集群部署
--------

由于模块之间具有一定的关联性，需按照以下顺序进行部署，避免服务依赖导致部署失败。

基础环境
::::::::

1. **支持平台**

    Linux

2. **依赖组件**

    `MongoDB <https://docs.mongodb.com/manual/tutorial/>`_

    `Kafka <https://kafka.apache.org/documentation/#basic_ops>`_

    `Consul <https://learn.hashicorp.com/tutorials/consul/get-started-install?in=consul/getting-started>`_ （建议每节点部署Consul agent）

启动clustermgr
::::::::::::::

部署clustermgr至少需要三个节点，以保证服务可用性。启动节点示例如下，节点启动需要更改对应配置文件，并保证集群节点之间的关联配置是一致。

1. 启动（三节点集群）

.. code-block:: bash

   nohup ./clustermgr -f clustermgr.conf
   nohup ./clustermgr -f clustermgr1.conf
   nohup ./clustermgr -f clustermgr2.conf

2. 三个节点的集群配置，示例节点一：``clustermgr.conf``

.. code-block:: json

   {
        "bind_addr":":9998",
        "cluster_id":1,
        "idc":["z0"], #机房列表
        "log": { # 系统日志
            "level": 0,
            "filename": "./run/cluster0.log"
        },
        "auditlog":{ # 审计日志
            "logdir": "/tmp/clustermgr/"
        },
        "region": "test-region-name",
        "normal_db_path":"/tmp/normaldb0",
        "normal_db_option": { # 自动创建目录
            "create_if_missing": true
        },
        "code_mode_policies": [ #编码模式策略
            {"code_mode":11,"min_size":0,"max_size":1024,"size_ratio":0.2,"enable":true}
        ],
        "volume_mgr_config":{ # 卷管理配置
            "volume_db_path":"/tmp/volumedb0",
            "volume_db_option": {
                "create_if_missing": true
            }
        },
        "cluster_config":{ # 集群配置
            "init_volume_num":100
        },
        "raft_config": {
            "raft_db_path": "/tmp/raftdb0",
            "raft_db_option": {
                "create_if_missing": true
            },
            "server_config": {
                "nodeId": 1,
                "listen_port": 10110,
                "raft_wal_dir": "/tmp/raftwal0",
                "peers": {"1":"127.0.0.1:10110","2":"127.0.0.1:10111","3":"127.0.0.1:10112"}
            },
            "raft_node_config":{
                "node_protocol": "http://",
                "nodes": {"1":"127.0.0.1:9998", "2":"127.0.0.1:9999", "3":"127.0.0.1:10000"}
            }
        },
        "disk_mgr_config":{
            "rack_aware":false,
            "host_aware":false
        }
   }

启动blobnode
::::::::::::

1. 在编译好的 ``blobnode`` 二进制目录下创建相关目录

.. code-block:: bash

   # 该目录对应配置文件的路径
   mkdir -p ./run/disks/disk{1..6} # 每个目录需要挂载磁盘，保证数据收集准确性
   mkdir -p ./run/auditlog

2. 启动服务

.. code-block:: bash

   nohup ./blobnode -f blobnode.conf

3. 示例 ``blobnode.conf``:

.. code-block:: json

   {
        "bind_addr": ":8899",
        "cluster": 1,
        "idc": "z0",
        "rack": "testrack",
        "host": "http://127.0.0.1:8899",  #ip替换为主机ip
        "disks": [ # 所需要创建目录结构
            {"path": "./run/disks/disk1", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk2", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk3", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk4", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk5", "auto_format": true,"max_chunks": 1024},
            {"path": "./run/disks/disk6", "auto_format": true,"max_chunks": 1024}
        ],
        "clustermgr": {
            "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
        },
        "disk_config":{
            "disk_reserved_space_B": 1,   # for debug
            "must_mount_point": true      # for debug
        },
        "flock_filename": "./run/blobnode.0.flock",
        "log":{ # 运行日志相关配置
            "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
            "filename": "./run/blobnode.log" # 运行日志文件，会自动轮转
        },
        "auditlog": {
            "logdir": "./run/auditlog"
        }
   }

启动allocator
:::::::::::::

部署allocator建议至少部署两个节点保证高可用。

1. 创建审计日志目录并启动服务

.. code-block:: bash

   mkdir /tmp/allocator
   nohup ./allocator -f allocator.conf

2. 示例 ``allocator.conf``:

.. code-block:: json

   {
        "bind_addr": ":9100",
        "host": "http://127.0.0.1:9100", #ip替换为主机ip
        "cluster_id": 1,
        "idc": "z0",
        "clustermgr": {
            "hosts": [
                "http://127.0.0.1:9998",
                "http://127.0.0.1:9999",
                "http://127.0.0.1:10000"
            ]
        },
        "log":{ # 运行日志相关配置
            "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
            "filename": "/tmp/allocator.log" # 运行日志文件，会自动轮转
        },
        "auditlog": {
            "logdir": "/tmp/allocator"
        }
   }

启动mqproxy
:::::::::::

1. 依赖kafka组件，需要提前创建blob_delete_topic、shard_repair_topic、shard_repair_priority_topic对应主题

.. code-block:: bash

   # 例如创建blob_delete_topic对应主题
   bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic blob_delete

2. 启动服务

.. code-block:: bash

   # 保证可用性，每个机房`idc`至少需要部署一个mqproxy节点
   nohup ./mqproxy -f mqproxy.conf

3. 示例 ``mqproxy.conf``:

.. code-block:: json

   {
        "bind_addr": ":9600", # 服务端口
        "cluster_id":1, # 集群id
        "clustermgr":{ # clustermgr服务地址
            "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
        },
        "mq":{
            "blob_delete_topic":"blob_delete", # 删除消息主题
            "shard_repair_topic":"shard_repair", # 修复消息主题
            "shard_repair_priority_topic":"shard_repair_prior", # 高优先级修复主题
            "msg_sender":{ # kafka地址
                "broker_list":["127.0.0.1:9092"]
            }
        },
        "service_register":{ # 自身服务注册信息
            "host":"http://127.0.0.1:9600", # 服务地址
            "idc":"z0"# 服务所属机房
        },
        "log":{ # 运行日志相关配置
          "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
          "filename": "/tmp/mqproxy.log" # 运行日志文件，会自动轮转
        },
        "auditlog": {# 审计日志相关配置
            "logdir": "./auditlog/mqproxy" # 审计日志目录
        }
   }

启动access
::::::::::

1. 启动服务

.. code-block:: bash

   # access模块为无状态单节点部署
   nohup ./access -f access.conf

2. 示例 ``access.conf``:

.. code-block:: json

   {
        "bind_addr": ":9500", # 服务端口
        "log": { # 运行日志相关配置
            "filename": "/tmp/access.log" # 运行日志文件
        },
        "auditlog": { # 审计日志相关配置
            "logdir": "./auditlog/access" # 审计日志目录
        },
        "consul_agent_addr": "127.0.0.1:8500", # 获取相关服务的consul地址
        "service_register": {
            "consul_addr": "127.0.0.1:8500", # access 服务注册地址
            "service_ip": "x.x.x.x" # access 服务IP
        },
        "stream": { # access server配置
            "idc": "z0", # access所在idc信息
            "cluster_config": { # cm 配置
                "region": "test-region" # region信息
            }
        }
   }

启动scheduler
:::::::::::::

1. 依赖mongodb，需要创建database.db_name、task_archive_store_db_name数据库

2. 启动服务

.. code-block:: bash

   nohup ./scheduler -f scheduler.conf

2. 示例 ``scheduler.conf``: 注意scheduler模块单节点部署

.. code-block:: json

   {
      "bind_addr": ":9800", # 服务端口
      "cluster_id": 1, # 集群id
      "clustermgr": { # clustermgr地址
        "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
      },
      "database": {# 后台任务相关配置
        "mongo": {
          "uri": "mongodb://127.0.0.1:27017" # mongodb 地址
        },
        "db_name": "scheduler" # 数据库名
      },
      "task_archive_store_db": {# 后台任务备份表
        "mongo": {
          "uri": "mongodb://127.0.0.1:27017" # mongodb 地址
        },
        "db_name": "task_archive_store" # 数据库名
      },
      "log":{ # 运行日志相关配置
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/scheduler.log" # 运行日志文件，会自动轮转
      },
      "auditlog": {# 审计日志相关配置
        "logdir": "./auditlog/scheduler" # 审计日志目录
      }
   }

启动worker
::::::::::

1. 启动服务

.. code-block:: bash

   # 每个机房`idc`至少部署一个worker节点
   nohup ./worker -f worker.conf

2. 示例 ``worker.conf``:

.. code-block:: json

   {
      "bind_addr": ":9910", # 服务端口
      "cluster_id": 1, # 集群id
      "service_register": { # 自身服务注册信息
        "host": "http://127.0.0.1:9910", # 服务地址
        "idc": "z0" # 服务所属机房
      },
      "scheduler": {# scheduler服务相关配置
        "host": "http://127.0.0.1:9800" # 服务地址
      },
      "dropped_bid_record": { # 丢弃blob id原因记录
        "dir": "./dropped" # 记录目录
      },
      "log":{ # 运行日志相关配置
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/worker.log" # 运行日志文件，会自动轮转
      },
      "auditlog": { # 审计日志相关配置
        "logdir": "./auditlog/worker" # 审计日志目录
      }
   }

启动tinker
::::::::::

1. 依赖kafka组件，需要提前创建shard_repair_conf.fail_topic_cfg.topic与viblob_delete_conf.fail_topic_cfg.topic

2. 依赖mongodb，需要创建数据库database_conf.db_name

3. 启动服务

.. code-block:: bash

   nohup ./tinker -f tinker.conf

4. 示例 ``tinker.conf``: 至少部署一个节点，配置消费kafka主题中的所有分区

.. code-block:: json

   {
      "bind_addr": ":9700", # 服务端口
      "cluster_id":1, # 集群id
      "database_conf": {# mongodb相关配置
          "mongo": {
            "uri": "mongodb://127.0.0.1:27017" # mongodb地址
          },
          "db_name": "tinker" # 数据库名
      },
      "shard_repair":{# 数据修补相关配置
           "broker_list":["127.0.0.1:9092"], # kafka 地址
           "priority_topics":[ # 修补主题配置
               {
                    "priority":1, # 修复优先级，数值越大优先级越高
                    "topic":"shard_repair", # 主题
                    "partitions":[0] # 消费分区
               },
               {
                   "priority":2, # 修复优先级，数值越大优先级越高
                   "topic":"shard_repair_prior", # 主题
                   "partitions":[0] # 消费分区
                }
           ],
           "fail_topic":{# 修补主题消费配置
                "topic":"shard_repair_failed", # 主题
                "partitions":[0] # 消费分区
           }
      },
      "blob_delete":{# 数据删除相关配置
            "broker_list":["127.0.0.1:9092"], # kafka地址
            "normal_topic":{ # 删除消息消费配置
                "topic":"blob_delete",# 主题
                "partitions":[0] # 消费分区
            },
            "fail_topic":{# 删除失败消息消费配置
                "topic":"fail_blob_delete", # 主题
                "partitions":[0] # 分区
            },
            "safe_delay_time_h":72, # 删除保护期
            "dellog":{ # 删除记录相关配置
                "dir": "./delete_log" # 删除日志目录
            }
      },
      "clustermgr": { # clustermgr地址
          "hosts": ["http://127.0.0.1:9998", "http://127.0.0.1:9999", "http://127.0.0.1:10000"]
       },
      "scheduler": {# scheduler服务地址
          "host": "http://127.0.0.1:9800"
      },
      "service_register":{ # 自身服务注册信息
          "host":"http://127.0.0.1:9700",# 服务地址
          "idc":"z0" # 服务所属机房
      },
      "log":{ # 运行日志相关配置
        "level":0,# 0:debug, 1:info, 2:warn, 3:error, 4:panic, 5:fatal
        "filename": "/tmp/tinker.log" # 运行日志文件，会自动轮转
      },
      "auditlog": {# 审计日志相关配置
        "logdir": "./auditlog/tinker" # 审计日志目录
      }
   }


配置说明
:::::::::

1. clustermgr
    1) code_mode_policies(编码模式策略)
    示例:

    .. code-block:: json

        {
           "code_mode" : 11 # 第11种编码策略, 具体策略方案，详见附录
           "min_size" : 0 # 最小上传对象大小为0
           "max_size" : 1024 # 最大上传对象大小为1024
           "size_ratio" : 1 # 不同策略的存储空间比列
           "enable" : true # 是否启用这个策略,ture代表启用，false不启用
        }


集群验证
--------

启动CLI
:::::::

在集群中任一台机器启动命名行工具 ``cli`` 后，设置access访问地址即可。

.. code-block:: bash

   ./cli # 启动cli 工具进入命名行

   # 用 config 命名 设置access访问地址
   $> config set Key-Access-PriorityAddrs http://127.0.0.1:9500

验证
::::

.. code-block:: bash

   # 上传文件，成功后会返回一个location，（-d 参数为文件实际内容）
   $> access put -v -d "test -data-"
   # 返回结果
   {"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}

   # 下载文件，用上述得到的location作为参数（-l），即可下载文件内容
   $> access get -v -l '{"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}'

   # 删除文件，用上述location作为参数（-l）；删除文件需要手动确认
   $> access del -v -l '{"cluster_id":1,"code_mode":10,"size":11,"blob_size":8388608,"crc":2359314771,"blobs":[{"min_bid":1844899,"vid":158458,"count":1}]}'


部署提示
--------

1. 对于clustermgr和blobnode部署失败后，重新部署需清理残留数据，避免注册盘失败或者数据显示错误，命令如下：

.. code-block:: bash

   # blobnode示例
   rm -f -r ./run/disks/disk*/.*
   rm -f -r ./run/disks/disk*/*

   # clustermgr示例
   rm -f -r /tmp/raftdb0
   rm -f -r /tmp/volumedb0
   rm -f -r /tmp/clustermgr
   rm -f -r /tmp/normaldb0
   rm -f -r /tmp/normalwal0

2. 所有模块部署成功后，上传验证需要延缓一段时间，等待创建卷成功。

附录
-----

1. 编码策略

.. csv-table:: 常用策略表
   :header: "类别", "编号", "描述"

   "EC15P12", "1", "{N: 15, M: 12, L: 0, AZCount: 3, PutQuorum: 24, GetQuorum: 0, MinShardSize: 2048}"
   "EC6P6", "2", "{N: 06, M: 06, L: 0, AZCount: 3, PutQuorum: 11, GetQuorum: 0, MinShardSize: 2048}"
   "EC16P20L2", "3", "{N: 16, M: 20, L: 2, AZCount: 2, PutQuorum: 34, GetQuorum: 0, MinShardSize: 2048}"
   "EC6P10L2", "4", "{N: 06, M: 10, L: 2, AZCount: 2, PutQuorum: 14, GetQuorum: 0, MinShardSize: 2048}"
   "EC12P4", "9", "{N: 12, M: 04, L: 0, AZCount: 1, PutQuorum: 15, GetQuorum: 0, MinShardSize: 2048}"
   "EC3P3", "11", "{N: 6, M: 3, L: 3, AZCount: 3, PutQuorum: 9, GetQuorum: 0, MinShardSize: 2048}"

*其中N: 数据块数量, M: 校验块数量, L: 本地校验块数量, AZCount: AZ数量,  PutQuorum: (N + M) / AZCount + N <= PutQuorum <= M + N， MinShardSize: 最小shard大小,将数据连续填充到 0-N 分片中，如果数据大小小于 MinShardSize*N，则与零字节对齐*，详见
`代码 <https://github.com/chubaofs/chubaofs/blobstore/common/codemode/codemode.go>`_
。