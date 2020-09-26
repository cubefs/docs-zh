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

设置节点状态
---------

.. code-block:: bash

   curl -v "http://192.168.0.11:17010/admin/setNodeState?start=6&end=10&nodeType=dataNode&state=true&zoneName=zone1"

功能：


设置节点（数据节点、元数据节点）状态，对于指定区域下的节点，id从start到end的节点，设置相应状态。
可以将其设置为不可写状态，以便于之后对数据节点、元数据节点进行下线操作


.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "start", "string", "开始id"
   "end", "string", "结束id"
   "nodeType", "string", "节点类型，dataNode：数据节点、metaNode：元数据节点、all：数据节点、元数据节点"
   "state", "string", "状态，设置为true，对应节点会变为不可写"
   "zoneName", "string", "指定区域，不可为空"


下线节点
---------

.. code-block:: bash

   curl -v "http://192.168.0.11:17010/dataNode/decommission?addr=192.168.0.33:17310&zoneName=default&strict=false"


从集群中下线某个数据节点, 该数据节点上的所有数据分片都会被异步的迁移到集群中其它可用的数据节点，分为普通模式和严格模式
为了避免下线node时其被写入新数据，可以先进行设置节点状态操作

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "数据节点和master的交互地址"
   "zoneName", "string", "指定区域，如果为空则默认值为原数据节点所属区域"
   "strict", "bool", "是否为严格模式，默认false"

严格模式，必须大小小于1G、文件数目相等，普通模式，大小差别小于1G即可

