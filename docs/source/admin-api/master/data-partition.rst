数据分片管理命令
=======================

创建
-------

.. code-block:: bash

   curl -v "http://127.0.0.1/dataPartition/create?count=400&name=test"


创建指定数量的数据分片

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "count", "int", "创建多少个数据分片"
   "name", "string", "卷的名字"

查询
-------

.. code-block:: bash

   curl -v "http://127.0.0.1/dataPartition/get?id=100"  | python -m json.tool


.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "id", "uint64", "数据分片的ID"

响应示例

.. code-block:: json

   {
       "PartitionID": 100,
       "LastLoadTime": 1544082851,
       "ReplicaNum": 3,
       "Status": 2,
       "Replicas": {},
       "PartitionType": "extent",
       "PersistenceHosts": {},
       "Peers": {},
       "MissNodes": {},
       "VolName": "test",
       "RandomWrite": true,
       "FileInCoreMap": {}
   }

下线副本
-------------

.. code-block:: bash

   curl -v "http://127.0.0.1/dataPartition/decommission?id=13&addr=127.0.0.1:5000"


移除数据分片的某个副本，并且创建一个新的副本

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "id", "uint64", "数据分片的ID"
   "addr", "string", "要下线的副本的地址"

比对副本文件
-------------

.. code-block:: bash

   curl -v "http://127.0.0.1/dataPartition/load?id=1"


给数据分片的每个副本都发送比对副本文件的任务，然后异步的检查每个副本上的文件crc是否一致

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "id", "uint64", "数据分片的ID"

磁盘下线
--------

.. code-block:: bash

   curl -v "http://127.0.0.1/disk/decommission?addr=127.0.0.1:5000&disk=/cfs1"


同步下线磁盘上的所有数据分片，并且为每一个数据分配在集群内创建一个新的副本

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "addr", "string", "要下线的副本的地址"
   "disk", "string", "故障磁盘"
