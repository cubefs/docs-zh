对象存储(ObjectNode)
==============================

如何部署对象存储服务
-------------------------------------------------------------------------

通过执行ChubaoFS的二进制文件并用“-c”参数指定的配置文件来启动一个ObjectNode进程。

.. code-block:: bash

   nohup cfs-server -c objectnode.json &


配置
-----------------------
对象管理节点使用 `JSON` 合适的配置文件


**属性**

.. csv-table::
   :header: "Key", "Type", "Description", "Mandatory"

   "role", "string", "Role of process and must be set to ``objectnode``", "Yes"
   "listen", "string", "
   | Listen and accept ip address and port of this server.
   | Format: ``IP:PORT`` or ``:PORT``
   | Default: ``:80``", "Yes"
   "region", "string", "
   | Region of this gateway. Used by S3-like interface signature validation.
   | Default: ``cfs_default``", "No"
   "domains", "string slice", "
   | Domain of S3-like interface which makes wildcard domain support
   | Format: ``DOMAIN``", "No"
   "logDir", "string", "Log directory", "Yes"
   "logLevel", "string", "
   | Level operation for logging.
   | Default: ``error``", "No"
   "masterAddr", "string slice", "
   | Format: ``HOST:PORT``.
   | HOST: Hostname, domain or IP address of master (resource manager).
   | PORT: port number which listened by this master", "Yes"
   "exporterPort", "string", "Port for monitor system", "No"
   "prof", "string", "Pprof port", "Yes"


**示例:**

.. code-block:: json

   {
        "role": "objectnode",
        "listen": "17410",
        "region": "cn_bj",
        "domains": [
            "object.cfs.local"
        ],
        "logDir": "/cfs/Logs/objectnode",
        "logLevel": "info",
        "masterAddr": [
	        "10.196.59.198:17010",
            "10.196.59.199:17010",
            "10.196.59.200:17010"
        ],
        "exporterPort": 9503,
        "prof": "7013"
   }

获取鉴权秘钥
----------------------------
鉴权秘钥由卷所有，并且存储于资源管理节点（master）的卷视图中

用户可以通过**Get Volume Information**来获取详细的管理接口信息，参加：:doc:`/admin-api/master/volume`

对象存储接口使用方法
-------------------------------
对象子系统（ObjectNode）提供S3兼容的对象存储接口，所以可以直接使用原生的Amazon S3 SDKs来使用系统。

通过 **Supported S3-compatible APIs** 获取更详细的信息，地址： :doc:`/design/objectnode`

通过 **Supported SDKs** 获取详细的SDK信息，地址： :doc:`/design/objectnode`
