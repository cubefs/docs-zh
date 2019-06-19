卷管理命令
===================

创建
----------

.. code-block:: bash

   curl -v "http://127.0.0.1/admin/createVol?name=test&capacity=100&owner=cfs&mpCount=3"


为用户创建卷，并分配一组数据分片和元数据分片.

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "name", "string", ""
   "capacity", "int", "卷的配额,单位是GB"
   "owner", "string", "vol的所有者"
   "mpCount","int","初始化metaPartition个数"

删除
-------------

.. code-block:: bash

   curl -v "http://127.0.0.1/vol/delete?name=test&authKey=md5(owner)"


首先把卷标记为逻辑删除, 然后通过周期性任务删除所有数据分片和元数据分片,最终从持久化存储中删除.

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "name", "string", ""
   "authKey", "string", "计算vol的所有者字段的MD5值作为认证信息"

查询
---------

.. code-block:: bash

   curl -v "http://127.0.0.1/client/vol?name=test&authKey=md5(owner)" | python -m json.tool


展示卷的基本信息,包括卷的名字,所有的数据分片和元数据分片信息等.

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "name", "string", ""
   "authKey", "string", "计算vol的所有者字段的MD5值作为认证信息"

响应示例

.. code-block:: json

   {
       "Name": "test",
       "VolType": "extent",
       "MetaPartitions": {},
       "DataPartitions": {}
   }


统计
-------

.. code-block:: bash

   curl -v http://127.0.0.1/client/volStat?name=test


展示卷的总空间大小和已使用空间大小信息

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "name", "string", ""

响应示例

.. code-block:: json

   {
       "Name": "test",
       "TotalSize": 322122547200000000,
       "UsedSize": 15551511283278
   }


更新
----------

.. code-block:: bash

   curl -v "http://127.0.0.1/vol/update?name=test&capacity=100&authKey=md5(owner)"

增加卷的配额

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"

   "name", "string", ""
   "capacity", "int", "卷的配额,单位是GB"
   "authKey", "string", "计算vol的所有者字段的MD5值作为认证信息"