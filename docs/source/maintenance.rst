运维手册
=================

编译构建
--------

编译服务端
^^^^^^^^^^^^^

使用如下命令同时构建server，client及相关的依赖：

.. code-block:: bash

   make build

如果构建成功，将在`build/bin` 目录中生成可执行文件`cfs-server`和`cfs-client`。

集群部署
----------

启动资源管理节点
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   nohup ./cfs-server -c master.json &


示例 ``master.json`` ：注意：master服务最少应该启动3个节点实例

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


   nohup ./cfs-server -c metanode.json &

示例 ``meta.json`` ：注意：metanode服务最少应该启动3个节点实例

.. code-block:: json

   {
       "role": "metanode",
       "listen": "9021",
       "prof": "9092",
       "logLevel": "info",
       "metadataDir": "/export/Data/metanode",
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

      nohup ./cfs-server -c datanode.json &

   示例 ``datanode.json`` :注意：datanode服务最少应该启动4个节点实例

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

   curl -v "http://192.168.31.173/admin/createVol?name=test&capacity=10000&owner=cfs"

   如果执行性能测试，请调用相应的API，创建足够多的数据分片（data partition）,如果集群中有8块磁盘，那么需要创建80个datapartition

挂载客户端
^^^^^^^^^^^^^

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
^^^^^^^^^^^^^
集群数据节点和元数据节点升级前，请先禁止集群自动为卷扩容数据分片.

1. 冻结集群

.. code-block:: bash

   curl -v "http://192.168.31.173/cluster/freeze?enable=true"

2. 升级节点

3. 开启自动扩容数据分片

.. code-block:: bash

   curl -v "http://192.168.31.173/cluster/freeze?enable=false"

集群管理
-----------

资源管理节点
^^^^^^^^^^^^^
1. 动态添加新节点
   假设原集群资源管理节点有三台，节点标识分别是1、2、3，现在要加入节点4。

    .. csv-table:: 机器列表
       :header: "ip:端口号","节点标识id"

       "192.168.31.173:80","1"
       "192.168.31.174:80","2"
       "192.168.31.175:80","3"
       "192.168.31.176:80","4"

   操作步骤如下

   1.1 执行资源管理添加节点的命令

        .. code-block:: bash

            curl -v "http://192.168.31.173/raftNode/add?addr=192.168.31.176:80&id=4"

   1.2 启动资源管理节点4

        .. code-block:: bash

           nohup ./cfs-server -c master.json &

2. 动态删除节点4

   1.1 执行资源管理添加节点的命令

        .. code-block:: bash

           curl -v "http://192.168.31.173/raftNode/remove?addr=192.168.31.176:80&id=4"

   1.2 关闭资源管理节点4

        .. code-block:: bash

           pkill cfs-server

3. 静态添加或删除节点
   直接修改所有节点的配置文件，peers项只保留想要的节点，重启所有资源管理节点即可

   .. code-block:: json

         {
           "peers": "1:192.168.31.173:80,2:192.168.31.174:80,3:192.168.31.175:80",
         }

元数据节点
^^^^^^^^^^^^^
1. 扩容新节点
   在新节点上编辑好配置文件，执行启动命令，详情请参考启动元数据节点章节

2. 下线节点
   为了保证有足够的副本冗余，如果出现机器故障，需要及时下线故障机器。
   执行元数据节点下线的命令，命令详情请参考 :doc:`admin-api/master/metanode`.

   .. code-block:: bash

            curl -v "http://127.0.0.1/metaNode/decommission?addr=127.0.0.1:9021"

数据节点
^^^^^^^^^^^^^
1. 扩容新节点
   在新节点上编辑好配置文件，执行启动命令，详情请参考启动数据节点章节

2. 下线节点
   为了保证有足够的副本冗余，如果出现机器故障，需要及时下线故障机器。
   执行数据节点下线的命令，命令详情请参考 :doc:`admin-api/master/datanode`.

   .. code-block:: bash

            curl -v "http://127.0.0.1/dataNode/decommission?addr=127.0.0.1:9021"

磁盘损坏处理
-------------
   磁盘损坏事件经常发生，当数据节点出现磁盘损坏时，要及时下线有问题的磁盘。
   具体操作步骤如下

   1. 执行磁盘下线命令

    .. code-block:: bash

       curl -v "http://127.0.0.1/disk/decommission?addr=127.0.0.1:5000&disk=/cfs1"

   2. 修改数据节点配置文件，重启数据节点

      2.1 从配置文件中删除故障磁盘，详细配置参数请参考 :doc:`user-guide/datanode`.

      2.2 关闭并重启数据节点

          .. code-block:: bash

                pkill cfs-server
                nohup ./cfs-server -c datanode.json &