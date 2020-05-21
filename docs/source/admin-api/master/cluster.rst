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
       "DataNodeStatInfo": {},
       "MetaNodeStatInfo": {},
       "VolStatInfo": {},
       "BadPartitionIDs": {},
       "BadMetaPartitionIDs": {},
       "MetaNodes": {},
       "DataNodes": {}
   }


冻结集群
--------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/cluster/freeze?enable=true"

如果启用了冻结集群功能,卷就不再自动地创建数据分片。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "enable", "bool", "如果设置为true，则集群被冻结"


获取集群空间信息
------------

.. code-block:: bash

    curl -v "http://10.196.59.198:17010/cluster/stat"

按区域展示集群的空间信息。

响应示例

.. code-block:: json

    {
        "DataNodeStatInfo": {
            "TotalGB": 1,
            "UsedGB": 0,
            "IncreasedGB": -2,
            "UsedRatio": "0.0"
        },
        "MetaNodeStatInfo": {
            "TotalGB": 1,
            "UsedGB": 0,
            "IncreasedGB": -8,
            "UsedRatio": "0.0"
        },
        "ZoneStatInfo": {
            "zone1": {
                "DataNodeStat": {
                    "TotalGB": 1,
                    "UsedGB": 0,
                    "AvailGB": 0,
                    "UsedRatio": 0,
                    "TotalNodes": 0,
                    "WritableNodes": 0
                },
                "MetaNodeStat": {
                    "TotalGB": 1,
                    "UsedGB": 0,
                    "AvailGB": 0,
                    "UsedRatio": 0,
                    "TotalNodes": 0,
                    "WritableNodes": 0
                }
            }
        }
    }

获取集群的拓扑信息
-----------------

.. code-block:: bash

    curl -v "http://10.196.59.198:17010/topo/get"

按区域展示集群的拓扑信息。

响应示例

.. code-block:: json

    [
        {
            "Name": "zone1",
            "Status": "available",
            "NodeSet": {
                "700": {
                    "DataNodeLen": 0,
                    "MetaNodeLen": 0,
                    "MetaNodes": [],
                    "DataNodes": []
                }
            }
        },
        {
            "Name": "zone2",
            "Status": "available",
            "NodeSet": {
                "800": {
                    "DataNodeLen": 0,
                    "MetaNodeLen": 0,
                    "MetaNodes": [],
                    "DataNodes": []
                }
            }
        }
    ]

更新可用区状态
-------------

.. code-block:: bash

    curl -v "http://10.196.59.198:17010/zone/update?name=zone1&enable=false"

更新可用区的状态为可用或不可用。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

    "name", "string", "可用区名称"
    "enable", "bool", "true表示可用，false为不可用"

获取所有可用区信息
----------------

.. code-block:: bash

    curl -v "http://10.196.59.198:17010/zone/list"

获取所有可用区的名称及可用状态。

响应示例

.. code-block:: json

    [
        {
            "Name": "zone1",
            "Status": "available",
            "NodeSet": {}
        },
        {
            "Name": "zone2",
            "Status": "available",
            "NodeSet": {}
        }
    ]