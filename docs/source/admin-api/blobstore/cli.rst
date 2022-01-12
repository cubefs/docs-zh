管理命令行工具
==============

使用命令行工具（CLI）可以实现方便快捷的集群管理。该工具为类unix命令行风格，可以查看集群的状态，实现了各模块的接口和功能。

目前CLI还不够完善，功能覆盖不到30%；通过我们不断完善，最终将会实现对于集群各模块接口功能的100%覆盖。

.. code-block:: txt

    1、命令行自动补全功能
    2、显示为可读数据类型


编译及配置
----------

通过 ``make cli`` 编译得到CLI工具。

``./cli -c cli.conf`` 启动命令行工具；其中 ``-c cli.conf`` 是可选配置项，主要配置一些常用变量，
比如access接入层服务发现地址，clustermgr服务地址等等。

.. code-block:: json

    {
        "access": {
            "consul_addr": "http://127.0.0.1:8500",
            "priority_addrs": [
                "http://localhost:9500",
                "http://127.0.0.1:9500"
            ]
        },
        "cm_addrs": [
            "http://localhost:9998",
            "http://127.0.0.1:9998"
        ],
        "verbose": true,
        "vverbose": false
    }


使用方法
--------

cli 可以作为普通命令，比如：

.. code-block:: bash

    cli MainCmd SubCmd [Cmd ...] [--flag Val ...] -- [-arg ...]

    1 #$> ./cli config set conf-key conf-val
    To set Key: conf-key Value: conf-val

    2 #$> ./cli util time
    timestamp = 1640156245364981202 (seconds = 1640156245 nanosecs = 364981202)
            --> format: 2021-12-22T14:57:25.364981202+08:00 (now)

``./cli`` 启动命令行。

.. code-block:: txt

    help 可以查看所有命令及简要说明
    建议使用`cmd subCmd ... --flag -- -arg` 方式传递参数

目前实现了部分模块的主要功能，如下：

.. csv-table::
   :header: "命令", "描述"

   "cli config", "管理该cli内存中的配置项"
   "cli util", "小工具集合，如解析location、解析时间、生成特定数据"
   "cli access", "文件的上传、下载、删除等"
   "cli cm", "集群信息查看和管理"
   "cli ...", "补充完善中 ......"


命令config
----------

.. code-block:: bash

    manager memory cache of config

    Usage:
      config [flags]

    Sub Commands:
      del   del config of keys
      get   get config in cache
      set   set config to cache
      type  print type in cache


命令util
--------

.. code-block:: bash

    util commands, parse everything

    Usage:
      util [flags]

    Sub Commands:
      location  parse location <[json | hex | base64]>
      redis     redis tools
      time      time format [unix] [format]
      token     parse token <token>
      vuid      parse vuid <vuid>


命令access
----------

.. code-block:: bash

    blobstore access api tools

    Usage:
      access [flags]

    Sub Commands:
      cluster  show cluster
      del      del file
      ec       show ec buffer size
      get      get file
      put      put file


命令cm
------

.. code-block:: bash

    cluster manager tools

    Usage:
      cm [flags]

    Sub Commands:
      config     config tools
      disk       disk tools
      listAllDB  list all db tools
      service    service tools
      stat       show stat of clustermgr
      volume     volume tools
      wal        wal tools
