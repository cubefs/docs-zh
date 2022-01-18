卷管理命令
===================

创建
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/createVol?name=test&capacity=100&owner=cfs&mpCount=3"


为用户创建卷，并分配一组数据分片和元数据分片. 在创建新卷时，默认分配10个数据分片和3个元数据分片。

ChubaoFS以 **Owner** 参数作为用户ID。在创建卷时，如果集群中没有与该卷的Owner同名的用户时，会自动创建一个用户ID为Owner的用户；如果集群中已存在用户ID为Owner的用户，则会自动将该卷的所有权归属于该用户。详情参阅： :doc:`/admin-api/master/user`

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需", "默认值"

   "name", "string", "卷名称", "是", "无"
   "volType", "int", "卷类型：0：副本卷，1：纠删码卷", "否", "0"
   "capacity", "int", "卷的配额,单位是GB", "是", "无"
   "owner", "string", "卷的所有者，同时也是用户ID", "是", "无"
   "mpCount", "int", "初始化元数据分片个数", "否", "3"
   "size", "int", "数据分片大小，单位GB", "否", "120"
   "followerRead", "bool", "允许从follower读取数据", "否", "false"
   "crossZone", "bool", "是否跨区域，如设为true，则不能设置zoneName参数", "否", "false"
   "zoneName", "string", "指定区域", "否", "如果crossZone设为false，则默认值为default"

删除
-------------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/delete?name=test&authKey=md5(owner)"


首先把卷标记为逻辑删除（status设为1）, 然后通过周期性任务删除所有数据分片和元数据分片,最终从持久化存储中删除。

在删除卷的同时，将会在所有用户的信息中删除与该卷有关的权限信息。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息"


查询卷详细信息
---------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/client/vol?name=test&authKey=md5(owner)" | python -m json.tool


展示卷的基本信息，包括卷的名字、所有的数据分片和元数据分片信息等。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息"

响应示例

.. code-block:: json

   {
       "Name": "test",
       "Owner": "user",
       "Status": "0",
       "FollowerRead": "true",
       "MetaPartitions": {},
       "DataPartitions": {},
       "CreateTime": 0
   }

查询卷数据分片详细信息
---------

.. code-block:: bash

   curl -v "http://192.168.0.12:17010/client/partitions?name=ltptest" | python -m json.tool


展示卷的所有的数据分片信息

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"

响应示例

.. code-block:: json

   {
       "Epoch": 0,
       "Hosts": [
           "192.168.0.34:17310",
           "192.168.0.33:17310",
           "192.168.0.32:17310"
       ],
       "IsRecover": false,
       "LeaderAddr": "192.168.0.33:17310",
       "PartitionID": 4,
       "ReplicaNum": 3,
       "Status": 2
   }


查询卷元数据分片详细信息
---------

.. code-block:: bash

   curl -v "http://192.168.0.12:17010/client/metaPartitions?name=ltptest" | python -m json.tool


展示卷的所有的元数据分片信息

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"

响应示例

.. code-block:: json

   {
       "DentryCount": 1,
       "End": 16777216,
       "InodeCount": 1,
       "IsRecover": false,
       "LeaderAddr": "192.168.0.23:17210",
       "MaxInodeID": 3,
       "Members": [
           "192.168.0.22:17210",
           "192.168.0.23:17210",
           "192.168.0.24:17210"
       ],
       "PartitionID": 1,
       "Start": 0,
       "Status": 2
   }


统计
-------

.. code-block:: bash

   curl -v http://10.196.59.198:17010/client/volStat?name=test


展示卷的总空间大小、已使用空间大小及是否开启读写token控制的信息。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"
   "version", "int", "卷版本，0：副本卷， 1：ec-卷，默认0-副本卷，访问ec卷必填"

响应示例

.. code-block:: json

   {
       "Name": "test",
       "TotalSize": 322122547200000000,
       "UsedSize": 155515112832780000,
       "UsedRatio": "0.48",
       "EnableToken": true
   }


更新
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/update?name=test&capacity=100&authKey=md5(owner)"

增加卷的配额，也可调整其它相关参数。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "name", "string", "卷名称", "是"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息", "是"
   "capacity", "int", "扩充后卷的配额,单位是GB", "是"
   "zoneName", "string", "更新后所在区域，若不设置将被更新至default区域", "是"
   "enableToken", "bool", "是否开启token控制读写权限，默认设为``false``", "否"
   "followerRead", "bool", "允许从follower读取数据", "否"

获取卷列表
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/list?keywords=test"

获取全部卷的列表信息，可按关键字过滤。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "keywords", "string", "获取卷名包含此关键字的卷信息", "否"

响应示例

.. code-block:: json

    [
       {
           "Name": "test1",
           "Owner": "cfs",
           "CreateTime": 0,
           "Status": 0,
           "TotalSize": 155515112832780000,
           "UsedSize": 155515112832780000
       },
       {
           "Name": "test2",
           "Owner": "cfs",
           "CreateTime": 0,
           "Status": 0,
           "TotalSize": 155515112832780000,
           "UsedSize": 155515112832780000
       }
    ]

