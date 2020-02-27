CSI插件支持
==============================
Chubaofs基于Container Storage Interface (CSI) (https://kubernetes-csi.github.io/docs/) 接口规范开发了CSI插件，以支持在Kubernetes集群中使用云存储。

.. csv-table::
  :header: "cfscsi", "kubernetes"

  "v0.3.0", "v1.12"
  "v1.0.0", "v1.15"

Kubernetes v1.12
-------------------

在Kubernetes v1.12集群中使用Chubaofs。

Kubernetes配置要求
^^^^^^^^^^^^^^^^^^^^^^^^

为了在kubernetes集群中部署cfscsi插件，kubernetes集群需要满足以下配置。

kube-apiserver启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true,MountPropagation=true
    --runtime-config=api/all

kube-controller-manager启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true

kubelet启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true,MountPropagation=true,KubeletPluginsWatcher=true
    --enable-controller-attach-detach=true

获取插件源码及脚本
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    $ git clone -b csi-spec-v0.3.0 https://github.com/chubaofs/chubaofs-csi.git
    $ cd chubaofs-csi

拉取官方CSI镜像
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    docker pull quay.io/k8scsi/csi-attacher:v0.3.0
    docker pull quay.io/k8scsi/driver-registrar:v0.3.0
    docker pull quay.io/k8scsi/csi-provisioner:v0.3.0

获取cfscsi镜像
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有两种方式可以实现。

* 从docker.io拉取镜像

.. code-block:: bash

    docker pull docker.io/chubaofs/cfscsi:v0.3.0

* 根据源码编译镜像

.. code-block:: bash

    make cfs-image

创建kubeconfig
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl create configmap kubecfg --from-file=pkg/cfs/deploy/kubernetes/kubecfg

创建RBAC和StorageClass
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl apply -f pkg/cfs/deploy/dynamic_provision/cfs-rbac.yaml
    kubectl apply -f pkg/cfs/deploy/dynamic_provision/cfs-sc.yaml

部署cfscsi插件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* 方式一：将cfscsi ControllerServer和NodeServer绑定在同一个sidecar容器

修改 ``pkg/cfs/deploy/dynamic_provision/sidecar/cfs-sidecar.yaml`` 文件，将环境变量 ``MASTER_ADDRESS`` 设置为Chubaofs的实际Master地址，将 ``<NodeServer IP>`` 设置为kubernetes集群任意IP（如果被调度到该IP的pod需要动态挂载Chubaofs网盘，则必须为该IP部署cfscsi sidecar容器）。

.. code-block:: bash

    kubectl apply -f pkg/cfs/deploy/dynamic_provision/sidecar/cfs-sidecar.yaml

* 方式二：将cfscsi插件ControllerServer和NodeServer分别部署为statefulset和daemonset（推荐此种）

修改 ``pkg/cfs/deploy/dynamic_provision/independent`` 文件夹下 ``csi-controller-statefulset.yaml`` 和 ``csi-node-daemonset.yaml`` 文件，将环境变量 ``MASTER_ADDRESS`` 设置为Chubaofs的实际Master地址 ，将 ``<ControllerServer IP>`` 设置为kubernetes集群中任意节点IP。

为Kubernetes集群中的节点添加标签，拥有 ``csi-role=controller`` 标签的节点为ControllerServer。拥有 ``csi-role=node`` 标签的节点为NodeServer，也可以删除 ``csi-node-daemonset.yaml`` 文件中的 ``nodeSelector`` ，这样kubernetes集群所有节点均为NodeServer。

.. code-block:: bash

    kubectl label nodes <ControllerServer IP> csi-role=controller
    kubectl label nodes <NodeServer IP1> csi-role=node
    kubectl label nodes <NodeServer IP2> csi-role=node
    ...

部署：

.. code-block:: bash

    kubectl apply -f pkg/cfs/deploy/dynamic_provision/independent/csi-controller-statefulset.yaml
    kubectl apply -f pkg/cfs/deploy/dynamic_provision/independent/csi-node-daemonset.yaml

创建PVC
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl apply -f pkg/cfs/deploy/dynamic_provision/cfs-pvc.yaml

nginx动态挂载Chubaofs示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    docker pull nginx
    kubectl apply -f pkg/cfs/deploy/dynamic_provision/pv-pod.yaml



Kubernetes v1.15
--------------------

在Kubernetes v1.15集群中使用Chubaofs。

Kubernetes配置要求
^^^^^^^^^^^^^^^^^^^^^^^^

为了在kubernetes集群中部署cfscsi插件，kubernetes集群需要满足以下配置。

kube-apiserver启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true,MountPropagation=true
    --runtime-config=api/all

kube-controller-manager启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true

kubelet启动参数:

.. code-block:: bash

    --feature-gates=CSIPersistentVolume=true,MountPropagation=true,KubeletPluginsWatcher=true
    --enable-controller-attach-detach=true

获取插件源码及脚本
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    $ git clone https://github.com/chubaofs/chubaofs-csi.git
    $ cd chubaofs-csi

拉取官方CSI镜像
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    docker pull quay.io/k8scsi/csi-attacher:v1.0.0
    docker pull quay.io/k8scsi/csi-node-driver-registrar:v1.0.2
    docker pull quay.io/k8scsi/csi-provisioner:v1.0.0

获取cfscsi镜像
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有两种方式可以实现。

* 方式一：从docker.io拉取镜像

.. code-block:: bash

    docker pull docker.io/chubaofs/cfscsi:v1.0.0

* 方式二：根据源码编译镜像

.. code-block:: bash

    make cfs-image

创建kubeconfig
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl create configmap kubecfg --from-file=pkg/chubaofs/deploy/kubernetes/kubecfg

创建RBAC和StorageClass
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/cfs-rbac.yaml
    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/cfs-sc.yaml

部署cfscsi插件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* 方式一：将cfscsi ControllerServer和NodeServer绑定在同一个sidecar容器

修改 ``pkg/chubaofs/deploy/dynamic_provision/sidecar/cfs-sidecar.yaml`` 文件，将环境变量 ``MASTER_ADDRESS`` 设置为Chubaofs的实际Master地址，将 ``<NodeServer IP>`` 设置为kubernetes集群任意IP（如果被调度到该IP的pod需要动态挂载Chubaofs网盘，则必须为该IP部署cfscsi sidecar容器）。

.. code-block:: bash

    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/sidecar/cfs-sidecar.yaml

* 方式二：将cfscsi插件ControllerServer和NodeServer分别部署为statefulset和daemonset（推荐此种）

修改 ``pkg/chubaofs/deploy/dynamic_provision/independent`` 文件夹下 ``csi-controller-statefulset.yaml`` 和 ``csi-node-daemonset.yaml`` 文件，将环境变量 ``MASTER_ADDRESS`` 设置为Chubaofs的实际Master地址 ，将 ``<ControllerServer IP>`` 设置为kubernetes集群中任意节点IP。

为Kubernetes集群中的节点添加标签，拥有 ``csi-role=controller`` 标签的节点为ControllerServer。拥有 ``csi-role=node`` 标签的节点为NodeServer，也可以删除 ``csi-node-daemonset.yaml`` 文件中的 ``nodeSelector`` ，这样kubernetes集群所有节点均为NodeServer。

.. code-block:: bash

    kubectl label nodes <ControllerServer IP> csi-role=controller
    kubectl label nodes <NodeServer IP1> csi-role=node
    kubectl label nodes <NodeServer IP2> csi-role=node
    ...

部署：

.. code-block:: bash

    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/independent/csi-controller-statefulset.yaml
    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/independent/csi-node-daemonset.yaml

创建PVC
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/cfs-pvc.yaml

nginx动态挂载Chubaofs示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    docker pull nginx
    kubectl apply -f pkg/chubaofs/deploy/dynamic_provision/pv-pod.yaml




