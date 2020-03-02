集群管理命令
===============

概述
--------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/getCluster" | python -m json.tool


展示集群基本信息，比如集群包含哪些数据节点和元数据节点，卷等。

响应示例

.. code-block:: json

   {
       "Name": "test",
       "LeaderAddr": "10.196.59.198:17010",
       "DisableAutoAlloc": false,
       "Applied": 225,
       "MaxDataPartitionID": 100,
       "MaxMetaNodeID": 3,
       "MaxMetaPartitionID": 1,
       "DataNodeStat": {},
       "MetaNodeStat": {},
       "VolStat": {},
       "MetaNodes": {},
       "DataNodes": {}
   }


冻结集群
--------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/cluster/freeze?enable=true"

如果启用了冻结集群功能,卷就不在自动的创建数据分片

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "enable", "bool", "如果设置为true，则集群被冻结"
