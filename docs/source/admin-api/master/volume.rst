卷管理命令
===================

创建
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/createVol?name=test&capacity=100&owner=cfs&mpCount=3"


为用户创建卷，并分配一组数据分片和元数据分片. 在创建新卷时，默认分配10个数据分片和3个元数据分片。

CubeFS以 **Owner** 参数作为用户ID。在创建卷时，如果集群中没有与该卷的Owner同名的用户时，会自动创建一个用户ID为Owner的用户；如果集群中已存在用户ID为Owner的用户，则会自动将该卷的所有权归属于该用户。详情参阅： :doc:`/admin-api/master/user`

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
   "cacheRuleKey", "string", "低频卷使用", "否", "非空时，匹配该字段的才会写入cache，空"
   "ebsBlkSize", "int", "每个块的大小，单位byte", "否", "默认8M"
   "cacheCap", "int", "低频卷 cache容量的大小,单位GB", "否", "低频卷必填"
   "cacheAction", "int", "低频卷写cache的场景，0-不写cache, 1-读数据回写cache, 2-读写数据都写到cache", "否", "0"
   "cacheThreshold", "int", "低频卷小于该值时，才写入到cahce中,单位byte", "否", "默认10M"
   "cacheTTL", "int", "低频卷cache淘汰时间，单位天", "否", "默认30"
   "cacheHighWater", "int", "低频卷cache淘汰的阈值，dp内容量淘汰上水位，达到该值时，触发淘汰", "否", "默认80，即120G*80/100=96G时，dp开始淘汰数据"
   "cacheLowWater", "int", "dp上容量淘汰下水位，达到该值时，不再淘汰，", "否", "默认60，即120G*60/100=72G，dp不再淘汰数据"
   "cacheLRUInterval", "int", "低容量淘汰检测周期，单位分钟", "否", "默认5分钟"

删除
-------------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/delete?name=test&authKey=md5(owner)"


首先把卷标记为逻辑删除（status设为1）, 然后通过周期性任务删除所有数据分片和元数据分片,最终从持久化存储中删除。ec卷使用大小为0时才能删除

在删除卷的同时，将会在所有用户的信息中删除与该卷有关的权限信息。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息"
   "forceDelVol", "bool", "是否强制删除卷，默认false"


查询卷详细信息
---------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/admin/getVol?name=test" | python -m json.tool


展示卷的基本信息，包括卷的名字、所有的数据分片和元数据分片信息等。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", "卷名称"

响应示例

.. code-block:: json

   {
       "Authenticate": false,
        "CacheAction": 0,
        "CacheCapacity": 0,
        "CacheHighWater": 80,
        "CacheLowWater": 60,
        "CacheLruInterval": 5,
        "CacheRule": "",
        "CacheThreshold": 10485760,
        "CacheTtl": 30,
        "Capacity": 10,
        "CreateTime": "2022-03-31 16:08:31",
        "CrossZone": false,
        "DefaultPriority": false,
        "DefaultZonePrior": false,
        "DentryCount": 0,
        "Description": "",
        "DomainOn": false,
        "DpCnt": 0,
        "DpReplicaNum": 16,
        "DpSelectorName": "",
        "DpSelectorParm": "",
        "FollowerRead": true,
        "ID": 706,
        "InodeCount": 1,
        "MaxMetaPartitionID": 2319,
        "MpCnt": 3,
        "MpReplicaNum": 3,
        "Name": "abc",
        "NeedToLowerReplica": false,
        "ObjBlockSize": 8388608,
        "Owner": "cfs",
        "PreloadCapacity": 0,
        "RwDpCnt": 0,
        "Status": 0,
        "VolType": 1,
        "ZoneName": "default"
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
       "CacheTotalSize": 0,
       "CacheUsedRatio": "",
       "CacheUsedSize": 0,
       "EnableToken": false,
       "InodeCount": 1,
       "Name": "abc-test",
       "TotalSize": 10737418240,
       "UsedRatio": "0.00",
       "UsedSize": 0
   }



更新
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/update?name=test&capacity=100&authKey=md5(owner)"

增加卷的配额，也可调整其它相关参数。

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "name", "string", "卷名称", "是"
   "description", "string", "卷描述信息", "否"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息", "是"
   "capacity", "int", "更新卷的datanode容量，单位G, 标准卷不能小于已使用容量", "否"
   "zoneName", "string", "更新后所在区域，若不设置将被更新至default区域", "是"
   "ebsBlkSize", "int", "低频卷的块大小，单位byte", "否"
   "followerRead", "bool", "允许从follower读取数据", "否"
   "cacheCap", "int", "低频卷cache容量大小", "否"
   "cacheAction", "int", "低频卷写cache的场景，0-不写cache, 1-读数据回写cache, 2-读写数据都写到cache", "否"
   "cacheThreshold", "int", "低频卷小于该值时，才写入到cahce中，单位byte", "否"
   "cacheTTL", "int", "低频卷cache淘汰时间，单位天", "否"
   "cacheHighWater", "int", "低频卷cache淘汰的阈值，dp内容量淘汰上水位，达到该值时，触发淘汰", "否"
   "cacheLowWater", "int", "dp上容量淘汰下水位，达到该值时，不再淘汰", "否"
   "cacheLRUInterval", "int", "容量淘汰周期，单位分钟", "否"
   "cacheRuleKey", "string", "修改cacheRule", "否"
   "emptyCacheRule", "bool", "是否置空cacheRule", "否"


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


扩容
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/expand?name=test&capacity=100&authKey=md5(owner) "

对指定卷进行扩容到指定容量

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "name", "string", "卷名称", "是"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息", "是"
   "capacity", "int", "扩充后卷的配额,单位是GB", "是"


缩容
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/vol/shrink?name=test&capacity=100&authKey=md5(owner) "

对指定卷进行缩小到指定容量

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "name", "string", "卷名称", "是"
   "authKey", "string", "计算vol的所有者字段的32位MD5值作为认证信息", "是"
   "capacity", "int", "压缩后卷的配额,单位是GB", "是"


预热卷
----------

.. code-block:: bash

   curl -v "http://10.196.59.198:17010/dataPartition/createPreLoad?name=test&cacheTTL=60&capacity=100 "

创建与热卷

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述", "是否必需"

   "name", "string", "卷名称", "是"
   "cacheTTL", "int", "预热数据的淘汰时间, 单位天, "是"
   "capacity", "int", "预热容量的大小,单位是GB", "是"
   "zoneName", "string", "预热数据的所属zone", "否"
