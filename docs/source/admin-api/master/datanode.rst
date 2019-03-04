数据节点管理命令
================

查询
-----

.. code-block:: bash

   curl -v "http://127.0.0.1/dataNode/get?addr=127.0.0.1:5000"  | python -m json.tool


显示数据节点的详情,包括数据节点的地址，总的容量，已使用空间等等.

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "数据节点和master的交互地址"

响应示例

.. code-block:: json

   {
       "MaxDiskAvailWeight": 3708923232256,
       "CreatedVolWeights": 2705829396480,
       "RemainWeightsForCreateVol": 36960383303680,
       "TotalWeight": 39666212700160,
       "UsedWeight": 2438143586304,
       "Available": 37228069113856,
       "Rack": "rack1",
       "Addr": "10.196.30.231:6000",
       "ReportTime": "2018-12-06T10:56:38.881784447+08:00",
       "Ratio": 0.06146650815226848,
       "SelectCount": 5,
       "Carry": 1.0655859145960367,
       "Sender": {
           "TaskMap": {}
       },
       "DataPartitionCount": 21
   }


下线节点
---------

.. code-block:: bash

   curl -v "http://127.0.0.1/dataNode/decommission?addr=127.0.0.1:5000"


从集群中下线某个数据节点, 该数据节点上的所有数据分片都会被异步的迁移到集群中其它可用的数据节点

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "数据节点和master的交互地址"