运维常见问题
============

1. 日志信息：metaPartition(xx) changeLeader to (xx)：

    切换leader。正常行为。

2. 日志信息：inode count is not equal, vol[xxx], mpID[xx]：

    inode数量不一致。因为写入数据时，三副本中只要有2个副本成功就算成功了，所以会存在三副本不一致的情况。查看日志了解具体原因。

3. 日志信息：clusterID[xxx] addr[xxx]_op[xx] has no response util time out：

    在进行[op]操作时，与该地址的节点连接超时。解决办法：检查网络连通性；查看服务进程是否还在。

4. 日志信息：checkFileCrcTaskErr clusterID[xxx] partitionID:xxx File:xxx badCrc On xxx:

    文件crc校验未通过。查看datanode日志了解具体原因。

5. master能否更换ip？

    可以，但除了调整Master的配置之外，还需要调整整个集群中所有功能性节点（MetaNode, DataNode, ObjectNode）以及Client的配置，让这些组件可以和调整IP后的Master正常通信。

6. 由三台master组成的集群中坏掉了一台，剩余两台重启能否正常提供服务？

    可以。由于Master使用了RAFT算法，在剩余节点数量超过总节点数量50%时，均可正常提供服务。

7. DataNode物理机故障怎么处理？

    可以下线故障节点，参考管理手册中DataNode下线API，链接：:doc:`/admin-api/master/datanode`，下线指定的DataNode节点。下线操作会将指定DataNode从所在集群的资源池中移除，然后ChubaoFS会自动使用集群中的可用资源补全下线DataNode引起的数据副本损失。

8. MetaNode物理机故障怎么处理?

    可以下线故障节点，参考管理手册中MetaNode下线API，链接：:doc:`/admin-api/master/metanode`，下线指定的MetaNode节点。下线操作会将指定MetaNode从所在集群的资源池中移除，然后ChubaoFS会自动使用集群中的可用资源补全下线MetaNode引起的元数据副本损失。