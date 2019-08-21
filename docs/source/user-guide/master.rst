资源管理节点
====================

Master负责管理ChubaoFS整个集群，主要存储5种元数据，包括：数据节点、元数据节点、卷、数据分片、元数据分片。所有的元数据都保存在master的内存中，并且持久化到RocksDB。
多个Master之间通过raft协议保证集群元数据的一致性。注意：master的实例最少需要3个

系统特性
---------------

- 多租户，资源隔离
- 多个卷共享数据节点和元数据节点,每个卷独享各自的数据分片和元数据分片
- 与数据节点和元数据节点即有同步交互，也有异步交互，交互方式与任务类型相关。

配置参数
--------------

ChubaoFS 使用 **JSON** 作为配置文件的格式.

.. csv-table:: 属性
   :header: "配置项", "类型", "描述", "是否必须"
   
   "role", "字符串", "进程的角色，值只能是 master", "是"
   "ip", "字符串", "主机ip", "是"
   "port", "字符串", "http服务监听的端口号", "是"
   "prof", "字符串", "golang pprof 端口号", "是"
   "id", "字符串", "区分不同的master节点", "是"
   "peers", "字符串", "raft复制组成员信息", "是"
   "logDir", "字符串", "日志文件存储目录", "是"
   "logLevel", "字符串", "日志级别，默认是 *error*.", "否"
   "retainLogs", "字符串", "保留多少条raft日志.", "是"
   "walDir", "字符串", "raft wal日志存储目录.", "是"
   "storeDir", "字符串", "RocksDB数据存储目录.此目录必须存在，如果目录不存在，无法启动服务", "是"
   "clusterName", "字符串", "集群名字", "是"
   "exporterPort", "整型", "The prometheus exporter port", "否"
   "consulAddr", "字符串", "consul注册地址，供prometheus exporter使用", "否"



**Example:**

.. code-block:: json

   {
    "role": "master",
    "ip": "127.0.0.1",
    "port": "8080",
    "prof":"10088",
    "id":"1",
    "peers": "1:127.0.0.1:8080,1:127.0.0.1:8081,1:127.0.0.1:8082",
    "retainLogs":"20000",
    "logDir": "/export/Logs/master",
    "logLevel":"info",
    "walDir":"/export/Data/master/raft",
    "storeDir":"/export/Data/master/rocksdbstore",
    "exporterPort": 9510,
    "consulAddr": "http://consul.prometheus-cfs.local",
    "clusterName":"test"
   }


启动服务
-------------

.. code-block:: bash

   nohup ./cfs-server -c master.json > nohup.out &
