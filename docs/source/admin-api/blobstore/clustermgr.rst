Clustermgr管理
===============

查看状态
--------

.. code-block:: bash

	curl "http://127.0.0.1:9998/stat" | python3 -m json.tool 

展示集群状态，包括raft状态、空间状态、卷信息统计等等。

响应示例

.. code-block:: json

   {
	    "leader_host": "127.0.0.1:9998",
	    "raft_status": {
		"nodeId": 1,
		"term": 1,
		"vote": 1,
		"commit": 5,
		"leader": 1,
		"raftState": "StateLeader",
		"applied": 5,
		"raftApplied": 5,
		"transferee": 0,
		"peers": [
		    {
			"id": 2,
			"host": "127.0.0.1:10111",
			"match": 5,
			"next": 6,
			"state": "ProgressStateReplicate",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    },
		    {
			"id": 3,
			"host": "127.0.0.1:10112",
			"match": 5,
			"next": 6,
			"state": "ProgressStateReplicate",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    },
		    {
			"id": 1,
			"host": "127.0.0.1:10110",
			"match": 5,
			"next": 6,
			"state": "ProgressStateProbe",
			"paused": false,
			"pendingSnapshot": 0,
			"active": true,
			"isLearner": false
		    }
		]
	    },
	    "space_stat": {
		"total_space": 0,
		"free_space": 0,
		"used_space": 0,
		"writable_space": 0,
		"total_blob_node": 0,
		"total_disk": 0,
		"disk_stat_infos": [
		    {
			"idc": "z0",
			"total": 0,
			"total_chunk": 0,
			"total_free_chunk": 0,
			"available": 0,
			"readonly": 0,
			"expired": 0,
			"broken": 0,
			"repairing": 0,
			"repaired": 0,
			"dropping": 0,
			"dropped": 0
		    }
		]
	    },
	    "volume_stat": {
		"total_volume": 0,
		"idle_volume": 0,
		"can_alloc_volume": 0,
		"active_volume": 0,
		"lock_volume": 0,
		"unlocking_volume": 0
	    }
   }
   

节点添加
---------

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1, "host": "127.0.0.1:9998", "member_type": 2}' "http://127.0.0.1:9998/member/add" 
   
添加节点，指定节点类型，地址和id。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "peer_id", "uint64", "raft节点id，不可重复"
   "host", "string", "主机地址"
   "member_type", "uint8", "节点类型，1表示leaner，2表示normal"
   
成员移除
--------

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1}' "http://127.0.0.1:9998/member/remove"

根据id移除节点。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "peer_id", "uint64", "raft节点id，不可重复"
   
切主
-----

.. code-block:: bash

   curl -X POST --header 'Content-Type: application/json' -d '{"peer_id": 1}' "http://127.0.0.1:9998/leadership/transfer"
   
根据id切换主节点。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "peer_id", "uint64", "raft节点id，不可重复"
   
启动或禁用后台任务
-----------------

.. csv-table::
   :header: "任务类型(type)", "任务名(key)", "开关(value)"

   "磁盘修复", "disk_repair", "Enable/Disable"
   "数据均衡", "balance", "Enable/Disable"
   "磁盘下线", "disk_drop", "Enable/Disable"
   "数据删除", "blob_delete", "Enable/Disable"
   "数据修补", "shard_repair", "Enable/Disable"
   "数据巡检", "vol_inspect", "Enable/Disable"
   
查看任务状态

.. code-block:: bash

   curl http://127.0.0.1:9998/config/get?key=balance

开启任务

.. code-block:: bash

   curl -X POST http://127.0.0.1:9998/config/set -d '{"key":"balance","value":"Enable"}' --header 'Content-Type: application/json'

关闭任务

.. code-block:: bash

   curl -X POST http://127.0.0.1:9998/config/set -d '{"key":"balance","value":"Disable"}' --header 'Content-Type: application/json'


