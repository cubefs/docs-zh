客户端
======

环境依赖
------------

- 插入内核FUSE模
- 安装libfuse

.. code-block:: bash

   modprobe fuse
   yum install -y fuse

配置文件
-------------------

fuse.json

.. code-block:: json

   {
     "mountPoint": "/mnt/fuse",
     "volName": "test",
     "owner": "cfs",
     "masterAddr": "192.168.31.173:80,192.168.31.141:80,192.168.30.200:80",
     "logDir": "/export/Logs/cfs",
     "logLevel": "info",
     "profPort": "10094"
   }

.. csv-table:: 配置选项
   :header: "名称", "类型", "描述", "必需"

   "mountPoint", "string", "挂载点", "是"
   "volName", "string", "集群名称", "是"
   "owner", "string", "所有者", "是"
   "masterAddr", "string", "Master节点地址", "是"
   "logDir", "string", "日志存放路径", "否"
   "logLevel", "string", "日志级别：debug, info, warn, error", "否"
   "profPort", "string", "golang pprof调试端口", "否"
   "exporterPort", "string", "监控模块端口", "否"
   "consulAddr", "string", "监控注册服务器地址", "否"
   "lookupValid", "string", "内核FUSE lookup有效期，单位：秒", "否"
   "attrValid", "string", "内核FUSE attribute有效期，单位：秒", "否"
   "icacheTimeout", "string", "客户端inode cache有效期，单位：秒", "否"
   "enSyncWrite", "string", "使能DirectIO同步写，即DirectIO强制数据节点落盘", "否"
   "autoInvalData", "string", "FUSE挂载使用AutoInvalData选项", "否"

挂载
---------------

执行如下命令挂载客户端：

.. code-block:: bash

   nohup ./client -c fuse.json &

如果使用示例的``fuse.json``，则客户端被挂载到``/mnt/fuse``。所有针对``/mnt/fuse``的操作都将被作用于CFS。
