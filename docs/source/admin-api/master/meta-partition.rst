元数据分片管理命令
========================

创建
---------

.. code-block:: bash

   curl -v "http://127.0.0.1/metaPartition/create?name=test&start=10000"


手动切分元数据分片,如果卷的最大的元数据分片inode的范围是[0,end),end 大于start参数,原来最大的元数据分片的inode范围变为[0,start], 新创建的元数据分片的范围是[start+1,end)

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "name", "string", "卷的名字"
   "start", "uint64", "根据此值切分元数据分片"

查询
-------

.. code-block:: bash

   curl -v "http://127.0.0.1/client/metaPartition?id=1" | python -m json.tool


展示元数据分片的详细信息,包括分片ID,分片的起始范围等等.

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "id", "uint64", "元数据分片ID"

响应示例

.. code-block:: json

   {
       "PartitionID": 1,
       "Start": 0,
       "End": 9223372036854776000,
       "MaxNodeID": 1,
       "Replicas": {},
       "ReplicaNum": 3,
       "Status": 2,
       "PersistenceHosts": {},
       "Peers": {},
       "MissNodes": {}
   }


下线副本
---------

.. code-block:: bash

   curl -v "http://127.0.0.1/metaPartition/decommission?id=13&addr=127.0.0.1:9021"


下线元数据分片的某个副本,并且创建一个新的副本

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "id", "uint64", "元数据分片ID"
   "addr", "string", "要下线副本的地址"

比对副本
--------

.. code-block:: bash

   curl -v "http://127.0.0.1/metaPartition/load?id=1"


发送比对副本任务到各个副本，然后检查各个副本的Crc是否一致

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "id", "uint64", "元数据分片ID"