CLI工具配置及使用方法
====================

使用命令行界面工具（CLI）可以实现方便快捷的集群管理。利用此工具，可以查看集群及各节点的状态，并进行各节点、卷及用户的管理。

随着CLI的不断完善，最终将会实现对于集群各节点接口功能的100%覆盖。

编译及配置
----------

下载ChubaoFS源码后，在 ``chubaofs/cli`` 目录下，执行命令 ``go build`` ，即可生成 ``cli`` 可执行程序。

同时，在 ``root`` 目录下会生成名为 ``.cfs-cli.json`` 的配置文件，修改master地址为当前集群的master地址即可。

使用方法
---------

在 ``chubaofs/cli`` 目录下，执行命令 ``./cli --help`` 或 ``./cli -h`` ，可获取CLI的帮助文档。

CLI共分为五类命令：

.. csv-table:: 命令列表
   :header: "命令", "描述"

   "cli cluster", "集群管理"
   "cli metanode", "元数据节点管理"
   "cli datanode", "数据节点管理"
   "cli volume, vol", "卷管理"
   "cli user", "用户管理"

集群管理命令
>>>>>>>>>>>>>

.. code-block:: bash

    ./cli cluster info     #获取集群信息，包括集群名称、地址、卷数量、节点数量及使用率等

元数据节点管理命令
>>>>>>>>>>>>>>>>>

.. code-block:: bash

    ./cli metanode list    #获取所有元数据节点的信息，包括id、地址、读写状态及存活状态

数据节点管理命令
>>>>>>>>>>>>>>>>>

.. code-block:: bash

    ./cli datanode list    #获取所有数据节点的信息，包括id、地址、读写状态及存活状态

卷管理命令
>>>>>>>>>>>>>>>>>

.. code-block:: bash

    ./cli volume create [VOLUME NAME] [USER ID] [flags]     #创建所有者是[USER ID]的卷[VOLUME NAME]
    Flags:
        --capacity uint                                     #指定卷的容量，单位GB（默认为10）
        --dp-size  uint                                     #指定数据分片的大小，单位GB（默认为120）
        --follower-read                                     #启用从follower副本中读取数据的功能（默认为true）
        --mp-count int                                      #指定初始元数据分片的数量（默认为3）
        --replicas int                                      #指定卷的副本数量（默认为3）
        -y, --yes                                           #跳过所有问题并设置回答为"yes"

.. code-block:: bash

    ./cli volume delete [VOLUME NAME] [flags]               #删除指定卷[VOLUME NAME]
    Flags:
        -y, --yes                                           #跳过所有问题并设置回答为"yes"

.. code-block:: bash

    ./cli volume info [VOLUME NAME] [flags]                 #获取卷[VOLUME NAME]的信息
    Flags:
        -d, --data-partition                                #显示数据分片的详细信息
        -m, --meta-partition                                #显示元数据分片的详细信息

.. code-block:: bash

    ./cli volume add-dp [VOLUME] [NUMBER]                   #创建并添加个数为[NUMBER]的数据分片至卷[VOLUME]

.. code-block:: bash

    ./cli volume list                                       #获取包含当前所有卷信息的列表

.. code-block:: bash

    ./cli volume transfer [VOLUME NAME] [USER ID] [flags]   #将卷[VOLUME NAME]转交给其他用户[USER ID]
    Flags：
        -f, --force                                         #强制转交
        -y, --yes                                           #跳过所有问题并设置回答为"yes"


用户管理命令
>>>>>>>>>>>>>>>>>

.. code-block:: bash

    ./cli user create [USER ID] [flags]         #创建用户[USER ID]
    Flags：
        --access-key string                     #指定用户用于对象存储功能的access key
        --secret-key string                     #指定用户用于对象存储功能的secret key
        --password string                       #指定用户密码
        --user-type string                      #指定用户类型，可选项为normal或admin（默认为normal）
        -y, --yes                               #跳过所有问题并设置回答为"yes"

.. code-block:: bash

    ./cli user delete [USER ID] [flags]         #删除用户[USER ID]
    Flags：
        -y, --yes                               #跳过所有问题并设置回答为"yes"

.. code-block:: bash

    ./cli user info [USER ID]                   #获取用户[USER ID]的信息

.. code-block:: bash

    ./cli user list                             #获取包含当前所有用户信息的列表

.. code-block:: bash

    ./cli user perm [USER ID] [VOLUME] [PERM]   #更新用户[USER ID]对于卷[VOLUME]的权限[PERM]
                                                #[PERM]可选项为"只读"（READONLY/RO）、"读写"（READWRITE/RW）、"删除授权"（NONE）

.. code-block:: bash

    ./cli user update [USER ID] [flags]         #更新用户[USER ID]的信息
    Flags：
        --access-key string                     #更新后的access key取值
        --secret-key string                     #更新后的secret key取值
        --user-type string                      #更新后的用户类型，可选项为normal或admin
        -y, --yes                               #跳过所有问题并设置回答为"yes"

