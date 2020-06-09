常见错误日志
============

1. 下线节点/分片副本：

- “no enough replicas”：目前该分片的存活副本数小于总副本数的一半，无法下线。
- ”live replicas num will be less than majority after offline nodeAddr“：节点下线后该分片的存活副本数小于总副本数的一半，无法下线。
- “err:cannot take the data replica offline, liveReplicas:”：剩余存活副本数少于总副本数一半。需要再启动一个副本。
- “action[hasMissingOneReplica],partitionID: ,err: there is a missing replica”：该分片的副本数比规定的副本数少。
- “vol[],meta partition[] is recovering,[] can't be decommissioned”：该元数据分片还在恢复中，无法下线。
- “vol[],data partition[] is recovering,[] can't be decommissioned”：该数据分片还在恢复中，无法下线。
- ”vol[],data partition[] can't decommision until it has recovered“：该数据分片还在恢复中，无法下线。
- “action[getAvailHosts] no enough writable hosts,replicaNum: MatchNodeCount:”：可写节点的个数小于要求的副本数量。
- “no node set available for creating a meta partition”：可用区没有其它可写的元数据节点。
- “no node set available for creating a data partition”：可用区没有其它可写的数据节点。
- “action[allocZonesForMetaNode],reqZoneNum[],candidateZones[],demandWriteNodes[],err:no zone available for creating a meta partition”：备选可用区不足，可用区内没有可写入的节点。
- “action[allocZonesForDataNode],reqZoneNum[],candidateZones[],demandWriteNodes[],err:no zone available for creating a data partition“：备选可用区不足，可用区内没有可写入的节点。
- “hasDownReplicasExcludePeer() too much,so donn't offline()”：down掉的副本太多，剩余副本少于一半，无法下线。使用``admin/getCluster``查看集群状态，查看BadPartitionIDs中的分片状态。当BadPartitionIDs为空时可以继续下线。
- “no leader”：当前partition没有选出主。

2. 创建卷：

- ”cluster has one zone,can't cross zone“：集群只有一个可用区，无法跨区申请。
- ”only the vol which don't across zones,can specified zoneName“：跨区域的卷不可以指定``zoneName``参数.
- ”action[createVol] initMetaPartitions failed,err:“：初始化元数据分片失败。查看元数据节点的状态是否可写。
- ”action[initMetaPartitions] vol[] init meta partition failed,mpCount[],expectCount[],err[]“：卷初始化的元数据分片与卷指定的元数据分片数``mpCount``不等。查看元数据节点的状态是否可写。
- “action[batchCreateDataPartition] after create [x] data partition,occurred error,err[]”：在创建第x个数据分片后创建数据分片失败。查看数据节点的状态。
- “action[getReplica],partitionID: xxx,locations: yyy,err: ”：在地址为“yyy”的节点上ID为“xxx”的分片副本获取失败。

3. 删除卷：

- “action[markDeleteVol] err[vol not exists]”：卷不存在。
- “client and server auth key do not match”：``authKey``的值不匹配。建议重新计算``md5(owner)``的值。

4. 更新卷：

- “capacity[] less than old capacity[]”：参数``capacity``必须大于或等于卷原有的容量。
- “don't support new replicaNum[] larger than old dpReplicaNum[]”：参数``replicaNum``不能大于卷原定的副本数。

5. 创建元数据分片（切分）：

- “action[updateInodeIDRange] vol[] partitionID[] nextStart[] to prevent overflow, not update end”：
- “action[updateInodeIDRange] vol[] id[] no leader”：该partition没有选出主。
- “action[updateEnd] partitionID[] err[]”：创建元数据分片失败。查看元数据节点的可写状态。

6. 磁盘下线：

- “node[] disk[] does not have any data partition”：该磁盘上没有数据分片信息。无需下线。

7. 其它：

- “raft status not health partitionID[]_nodeID[]_leader[]_state[]_replicas[]”：该节点负载较高。查看各节点是否可写。
- ”[deleteExtentsFromList] partitionId= , xxx file corrupted“：删除”xxx“文件。
- “raft partitionID[xxx] replicaID[yyy] not active peer[]”：ID为“xxx”的分片副本状态异常。使用
