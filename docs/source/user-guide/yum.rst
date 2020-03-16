yum工具自动部署集群
==================

可以使用yum工具在CentOS 7+操作系统中快速部署和启动ChubaoFS集群. 该工具的rpm依赖项可通过以下命令安装:

.. code-block:: bash

    $ yum install http://storage.jd.com/chubaofsrpm/latest/cfs-install-latest-el7.x86_64.rpm
    $ cd /cfs/install
    $ tree -L 2
    .
    ├── install_cfs.yml
    ├── install.sh
    ├── iplist
    ├── src
    └── template
        ├── client.json.j2
        ├── create_vol.sh.j2
        ├── datanode.json.j2
        ├── grafana
        ├── master.json.j2
        └── metanode.json.j2

可根据你的实际情况，在 **iplist** 文件中修改ChubaoFS集群的参数.

- **[master]** , **[datanode]** , **[metanode]** , **[monitor]** , **[client]** 模块定义了每个模块所有节点的IP地址。

- **#datanode config** 模块定义了每个DataNode的启动参数，其中 **datanode_disks** 参数代表数据信息 **存储路径** 和 **保留空间** （Byte）并用":"分隔开。存储路径需要根据你的物理机实际情况修改，确保每台DataNode上该路径存在且有至少30GB可用空间；保留空间是指始终为该路径保留的最小空闲空间（Bytes）。

- **[cfs:vars]** 模块定义了所有节点的ssh登陆信息，需要事先将集群中所有节点的登录名和密码进行统一。

- **#metanode config** 模块定义了MetaNode的启动参数，其中 **metanode_totalMem** 参数代表MetaNode进程可占用机器的最大内存（Bytes）

.. code-block:: yaml

    [master]
    10.196.59.198
    10.196.59.199
    10.196.59.200
    [datanode]
    ...
    [cfs:vars]
    ansible_ssh_port=22
    ansible_ssh_user=root
    ansible_ssh_pass="password"
    ...
    #datanode config
    ...
    datanode_disks =  '"/data0:10737418240","/data1:10737418240"'
    ...
    #metanode config
    ...
    metanode_totalMem = "28589934592"
    ...

更多配置介绍请参考 :doc:`master`; :doc:`metanode`; :doc:`datanode`; :doc:`client`; :doc:`monitor` 。

用 **install.sh** 脚本启动ChubaoFS集群，并确保首先启动Master。

.. code-block:: bash

    $ bash install.sh -h
    Usage: install.sh [-r --role datanode or metanode or master or monitor or client or all ] [-v --version 1.5.1 or latest]
    $ bash install.sh -r master
    $ bash install.sh -r metanode
    $ bash install.sh -r datanode
    $ bash install.sh -r monitor
    $ bash install.sh -r client

全部角色启动后，可以登录到 **client** 角色所在节点验证挂载点 **/cfs/mountpoint** 是否已经挂载ChubaoFS文件系统。

在浏览器中打开链接http://consul.prometheus-cfs.local 查看监控系统(监控系统的IP地址已在 **iplist** 文件的 **[monitor]** 模块定义).
