数据节点管理命令
================

新增
---------

.. code-block:: bash

   curl -v "http://192.168.0.11:17010/dataNode/add?addr=192.168.0.33:17310&zoneName=default"


在对应区域上添加新的数据节点

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "addr", "string", "数据节点和master的交互地址"
   "zoneName", "string", "指定区域，如果为空则默认值为default"


查询
-----

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/dataNode/get?addr=10.196.59.201:17310"  | python -m json.tool


显示数据节点的详情，包括数据节点的地址、总的容量、已使用空间等等。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "数据节点和master的交互地址"

响应示例

.. code-block:: json

   {
       "TotalWeight": 39666212700160,
       "UsedWeight": 2438143586304,
       "AvailableSpace": 37228069113856,
       "ID": 2,
       "Zone": "zone1",
       "Addr": "10.196.59.201:17310",
       "ReportTime": "2018-12-06T10:56:38.881784447+08:00",
       "IsActive": true
       "UsageRatio": 0.06146650815226848,
       "SelectTimes": 5,
       "Carry": 1.0655859145960367,
       "DataPartitionReports": {},
       "DataPartitionCount": 21,
       "NodeSetID": 3,
       "PersistenceDataPartitions": {},
       "BadDisks": {}
   }


下线节点
---------

.. code-block:: bash

   curl -v "http://192.168.0.11:17010/dataNode/decommission?addr=192.168.0.33:17310"


从集群中下线某个数据节点, 该数据节点上的所有数据分片都会被异步的迁移到集群中其它可用的数据节点

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "数据节点和master的交互地址"
