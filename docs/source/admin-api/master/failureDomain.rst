故障域配置及管理命令
================

一、升级及配置项
---------------------------
Cluster级别配置
---------------------------
默认是要支持原有的配置，启用故障域需要一个特殊的配置，增加cluster级别的配置，是否支持故障域

FaultDomain               bool  // 默认false

否则无法区分，新增zone是故障域zone还是归属于原有cross_zone

Volume级别配置
---------------------------
保留：

crossZone        bool  //跨zone

新增：

default_priority为true在生效，优先选择原有的zone，而不是从故障域里面分配



故障域zone识别
---------------------------
1. cluster配置故障域选项，如果此时master重启，zoneset重新加载，Putnodeset到group里面，如何判断nodeset是属于故障域呢？？

2. 如果cluster不配置，直接添加zone，cluster如何区分zone属于故障域？

3. 全部掉电，新增zone的机器和普通的机器无法区分，除了部分已经持久化


解决方案：

1. 配置当前master为crosszone，master重启，之后再添加新的zone；

2. 重启为了将当前的zone持久化为非故障域zone（持久化层没有该信息，默认当前zone应该全部持久化为旧的zone（default zone））；

3. 重启后加载，后面添加新的zone，则默认为新的故障域zone；并持久化；


二、配置小结
---------------------------
1. 现有的cluster，无论是自建的，还是社区的，无论是单个zone，还是跨zone，如果需要故障域启用，需要cluster支持，master重启，配置更新，同时管控更新现有volume的策略。否则继续沿用原有策略。

2. 如果cluster支持，volome不选择使用，则继续原有volome策略，需要在原有zone中按原有策略分配。原有资源耗尽再使用新的zone资源，

3. 如果cluster不支持，volome无法自己启用的故障域策略


=========================  =========================  ======================  ===================================================================================
  Cluster:faultDomain           Vol:crossZone           Vol:normalZonesFirst     Rules for volume to use domain
=========================  =========================  ======================  ===================================================================================
N                                  N/A                        N/A                     Do not support domain
Y                                  N                          N/A               Write origin resources first before fault domain until origin reach threshold
Y                                  Y                          N                       Write fault domain only
Y                                  Y                          Y                 Write origin resources first before fault domain until origin reach threshold
=========================  =========================  ======================  ===================================================================================


三、管理命令
---------------------------
创建使用故障域的volume
---------

.. code-block:: bash

      curl "http://192.168.0.11:17010/admin/createVol?name=volDomain&capacity=1000&owner=cfs&crossZone=true&normalZonesFirst=true"


.. csv-table:: 参数列表
   :header: "参数", "类型", "描述"
   
   "crossZone", "string", "是否跨zone"
   "normalZonesFirst", "非故障域优先", ""


查看故障域使用情况
---------
.. code-block:: bash

      curl -v  "http://192.168.0.11:17010/admin/getDomainInfo"


更新故障域数据使用上限
---------
.. code-block:: bash

      curl "http://192.168.0.11:17010/admin/updateDomainDataRatio?ratio=0.7"
      
      
查看非故障域数据使用上限
---------
.. code-block:: bash

      curl "http://192.168.0.11:17010/admin/updateZoneExcludeRatio?ratio=0.7"
