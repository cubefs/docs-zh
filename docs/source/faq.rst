常见问题
================

1. undefined reference to 'ZSTD_versionNumber' 类似问题

  可以使用下面两种方式解决
  
  a. CGO_LDFLAGS添加指定库即可编译，
  
     例如：``CGO_LDFLAGS="-L/usr/local/lib -lrocksdb -lzstd"`` 这种方式，要求其他部署机器上也要安装 ``zstd`` 库

  b. 删除自动探测是否安装zstd库的脚本

     文件位置示例: rockdb-5.9.2/build_tools/build_detect_platform
     
     删除的内容如下
     
     .. code-block:: bash
     
        # Test whether zstd library is installed
            $CXX $CFLAGS $COMMON_FLAGS -x c++ - -o /dev/null 2>/dev/null  <<EOF
              #include <zstd.h>
              int main() {}
        EOF
            if [ "$?" = 0 ]; then
                COMMON_FLAGS="$COMMON_FLAGS -DZSTD"
                PLATFORM_LDFLAGS="$PLATFORM_LDFLAGS -lzstd"
                JAVA_LDFLAGS="$JAVA_LDFLAGS -lzstd"
            fi
 
2. 本机编译ChubaoFS，部署到其它机器上无法启动

    首先请确认使用 ``PORTABLE=1 make static_lib`` 命令编译rocksdb，然后使用ldd命令查看依赖的库，在机器上是否安装，安装缺少的库后，执行 ``ldconfig`` 命令

3. MetaNode的状态为”只读“是什么原因？

    - 原因1：长时间未接收到MetaNode的心跳。

    - 原因2：MetaNode的使用率达到了设定的阈值。

    - 原因3：MetaNode的剩余可用内存（ ``TotalWeight`` 指标减去 ``UsedWeight`` 指标），小于或等于master节点的配置 ``metaNodeReservedMem`` 值。可以适当调高MetaNode的 ``TotalWeight`` 配置参数。

    - 原因4：MetaNode的元数据分片数量达到了默认的最大分片数量（10000）。

4. 创建卷时指定的参数，如果日后需要变更怎么办？

    使用master的updateVol接口可以实现变更容量 ``capacity`` 、区域 ``zoneName`` 、开启token控制 ``enableToken`` 、 ``followerRead`` 等参数。

5. 如何上线新磁盘？

    将新磁盘的路径加入 ``DataNode`` 的配置参数 ``disks`` 中，然后重启节点即可。
