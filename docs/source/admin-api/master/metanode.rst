元数据节点管理命令
=====================

新增
---------

.. code-block:: bash

   curl -v "http://192.168.0.11:17010/metaNode/add?addr=192.168.0.33:17310"


在对应区域上添加新的元数据节点

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "addr", "string", "元数据节点和master的交互地址"


查询
-----

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/metaNode/get?addr=10.196.59.202:17210"  | python -m json.tool


展示元数据节点的详细信息，包括地址、总的内存大小、已使用内存大小等等。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "addr", "string", "元数据节点和master的交互地址"

响应示例

.. code-block:: json

   {
       "ID": 3,
       "Addr": "10.196.59.202:17210",
       "IsActive": true,
       "Zone": "zone1",
       "MaxMemAvailWeight": 66556215048,
       "TotalWeight": 67132641280,
       "UsedWeight": 576426232,
       "Ratio": 0.008586377967698518,
       "SelectCount": 0,
       "Carry": 0.6645600532184904,
       "Threshold": 0.75,
       "ReportTime": "2018-12-05T17:26:28.29309577+08:00",
       "MetaPartitionCount": 1,
       "NodeSetID": 2,
       "PersistenceMetaPartitions": {}
   }


下线节点
--------

.. code-block:: bash

  curl -v "http://10.196.59.198:17010/metaNode/decommission?addr=10.196.59.202:17210" 

为了避免下线node时其被写入新数据，可以先进行设置节点状态操作
从集群中下线某个元数据节点, 该节点上的所有元数据分片都会被异步的迁移到集群中其它可用的元数据节点，分为普通模式和严格模式。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "addr", "string", "元数据节点和master的交互地址"


设置阈值
---------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/threshold/set?threshold=0.75"


如果某元数据节点内存使用率达到这个阈值，则该节点上所有的元数据分片都会被设置为只读。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "threshold", "float64", "元数据节点能使用本机内存的最大比率"


迁移
---------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/metaNode/migrate?srcAddr=src&targetAddr=dst&count=3"


从源元数据节点迁移指定个数元数据分区至目标元数据节点。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "srcAddr", "string", "迁出元数据节点地址"
   "targetAddr", "string", "迁入元数据节点地址"
   "count", "int", "迁移元数据分区的个数，非必填，默认15个"
