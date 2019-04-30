快速开始
=================

编译构建
--------

编译服务端
^^^^^^^^^^^^^

在ChubaoFS的设计中，服务端包括资源管理节点（resource manager），元数据节点（metanode）和数据节点（datanode）。为了版本匹配及混合部署便捷，服务端提供同一个可执行程序。

构建ChubaoFS服务端需要依赖RocksDB， `build RocksDB v5.9.2+ <https://github.com/facebook/rocksdb/blob/master/INSTALL.md>`_ 。推荐使用 ``make static_lib`` 编译RocksDB。然后执行如下命令编译ChubaoFS客户端。

.. code-block:: bash

   cd cmd; sh build.sh

编译客户端
^^^^^^^^^^^^
.. code-block:: bash

   cd client; sh build.sh

集群部署
----------

启动资源管理节点
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   nohup ./cmd -c master.json &


示例 ``master.json`` ：

.. code-block:: json

   {
     "role": "master",
     "ip": "192.168.31.173",
     "port": "80",
     "prof":"10088",
     "id":"1",
     "peers": "1:192.168.31.173:80,2:192.168.31.141:80,3:192.168.30.200:80",
     "retainLogs":"20000",
     "logDir": "/export/Logs/master",
     "logLevel":"info",
     "walDir":"/export/Data/master/raft",
     "storeDir":"/export/Data/master/rocksdbstore",
     "warnLogDir":"/export/home/tomcat/UMP-Monitor/logs/",
     "consulAddr": "http://consul.prometheus-cfs.local",
     "exporterPort": 9510,
     "clusterName":"cfs"
   }

   
详细配置参数请参考 :doc:`user-guide/master` 。

启动元数据节点
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   nohup ./cmd -c meta.json &

示例 ``meta.json`` ：

.. code-block:: json

   {
       "role": "metanode",
       "listen": "9021",
       "prof": "9092",
       "logLevel": "info",
       "metaDir": "/export/Data/metanode",
       "logDir": "/export/Logs/metanode",
       "raftDir": "/export/Data/metanode/raft",
       "raftHeartbeatPort": "9093",
       "raftReplicaPort": "9094",
       "totalMem":  "17179869184",
       "consulAddr": "http://consul.prometheus-cfs.local",
        "warnLogDir":"/export/home/tomcat/UMP-Monitor/logs/",
       "exporterPort": 9511,
       "masterAddrs": [
           "192.168.31.173:80",
           "192.168.31.141:80",
           "192.168.30.200:80"
       ]
   }


详细配置参数请参考 :doc:`user-guide/metanode`.

启动数据节点
^^^^^^^^^^^^^^

1. 准备数据目录

   **推荐** 使用单独磁盘作为数据目录，配置多块磁盘能够达到更高的性能。

   **磁盘准备**

    1.1 查看机器磁盘信息，选择给ChubaoFS使用的磁盘

        .. code-block:: bash
   
           fdisk -l
	
    1.2 格式化磁盘，建议格式化为XFS

        .. code-block:: bash
   
           mkfs.xfs -f /dev/sdx
		
    1.3 创建挂载目录
        
        .. code-block:: bash
   
           mkdir /data0	
	
    1.4 挂载磁盘
        
        .. code-block:: bash
   
           mount /dev/sdx /data0

2. 启动数据节点

   .. code-block:: bash
   
      nohup ./cmd -c datanode.json &

   示例 ``datanode.json`` :
   
   .. code-block:: json

      {
        "role": "datanode",
        "port": "6000",
        "prof": "6001",
        "logDir": "/export/Logs/datanode",
        "logLevel": "info",
        "raftHeartbeat": "9095",
        "raftReplica": "9096",
        "consulAddr": "http://consul.prometheus-cfs.local",
        "warnLogDir":"/export/home/tomcat/UMP-Monitor/logs/",
        "exporterPort": 9512,
        "masterAddr": [
        "192.168.31.173:80",
        "192.168.31.141:80",
        "192.168.30.200:80"
        ],
        "rack": "",
        "disks": [
           "/data0:21474836480",
           "/data1:21474836480"
       ]
      }

   详细配置参数请参考 :doc:`user-guide/datanode`.

创建Volume卷
^^^^^^^^^^^^^

.. code-block:: bash

   curl -v "http://192.168.31.173/admin/createVol?name=test&capacity=100&owner=cfs"

   如果执行性能测试，请调用相应的API，创建足够多的数据分片（data partition）

挂载客户端
------------

1. 运行 ``modprobe fuse`` 插入FUSE内核模块。
2. 运行 ``yum install -y fuse`` 安装libfuse。
3. 运行 ``nohup client -c fuse.json &`` 启动客户端。

   样例 *fuse.json* ,
   
   .. code-block:: json
   
      {
        "mountPoint": "/mnt/fuse",
        "volName": "test",
		"owner": "cfs",
        "masterAddr": "192.168.31.173:80,192.168.31.141:80,192.168.30.200:80",
        "logDir": "/export/Logs/client",
        "warnLogDir":"/export/home/tomcat/UMP-Monitor/logs/",
        "profPort": "10094",
        "logLevel": "info"
      }


详细配置参数请参考 :doc:`user-guide/client`.

升级注意事项
---------------
集群数据节点和元数据节点升级前，请先禁止集群自动为卷扩容数据分片.

1. 冻结集群

.. code-block:: bash

   curl -v "http://192.168.31.173/cluster/freeze?enable=true"

2. 升级节点

3. 开启自动扩容数据分片

.. code-block:: bash

   curl -v "http://192.168.31.173/cluster/freeze?enable=false"
