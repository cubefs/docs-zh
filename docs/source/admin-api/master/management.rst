资源管理命令
=================

增加
-----

.. code-block:: bash

   curl -v "http://127.0.0.1/raftNode/add?addr=127.0.0.1:80&id=3"


增加新的master节点到raft复制组

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "master的ip地址, 格式为ip:port"
   "id", "uint64", "master的节点标识"

删除
---------

.. code-block:: bash

   curl -v "http://127.0.0.1/raftNode/remove?addr=127.0.0.1:80&id=3"


从raft复制组总移除某个节点

.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "addr", "string", "master的ip地址, 格式为ip:port"
   "id", "uint64", "master的节点标识"