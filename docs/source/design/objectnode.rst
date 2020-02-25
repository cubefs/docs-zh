对象存储 (ObjectNode)
=============================

对象存储系统提供兼容S3的对象存储接口。它使得ChubaoFS成为一个可以将两种通用类型接口进行融合的存储（POSIX和S3兼容接口）。可以使用户使用原生的Amazon S3 SDK操作ChubaoFS中的文件。

框架
---------

  .. image:: ../pic/cfs-object-subsystem-structure.png
     :align: center

ObjectNode是一个功能性的子系统节点。它根据需要从资源管理器（Master）获取卷视图（卷拓扑）。
每个ObjectNode直接与元数据子系统（MetaNode）和数据子系统（DataNode）通信。

ObjectNode是一种无状态设计，具有很高的可扩展性，能够直接操作ChubaoFS集群中存储的所有文件，而无需任何卷装入操作。

特性
--------

- 支持原生的Amazon S3 SDKs的对象存储接口
- 支持两种通用接口的融合存储（POSIX和S3兼容接口）
- 无状态和高可靠性 

语义转换
-------------------
基于原有POSIX兼容性的设计。每个来自对象存储接口的文件操作请求都需要对POSIX进行语义转换。

.. csv-table::
    :header: "POSIX", "Object Storage"

    "``Volume``", "``Bucket``"
    "``Path``", "``Key``"

**示例:**

      .. image:: ../pic/cfs-object-subsystem-semantic.png
        :align: center

    Put object '*example/a/b.txt*' will be create and write data to file '*/a/b.txt*' in volume '*example*'.

鉴权
--------------
对象存储接口中的签名验证算法与amazons3服务完全兼容。由资源管理器（Master）在创建卷时生成的 *AccessKey* 和 *SecretKey* 组成的身份验证， *AccessKey* 是整个ChubaoFS集群中唯一的16个字符的字符串。

由卷拥有并由资源管理器（主）与卷视图（卷拓扑）一起存储的身份验证密钥。

用户可以通过管理API获取，请参见 **Get Volume Information** ，链接： :doc:`/admin-api/master/volume`

临时隐藏数据
-------------------------
以原子方式在对象存储接口中进行写操作。每个写操作都将创建数据并将其写入一个不可见的临时对象。ObjectNode中的volume运算符将文件数据放入临时文件，临时文件的元数据中只有'**inode**'而没有'**dentry**'。当所有文件数据都成功存储时，volume操作符在元数据中创建或更新'**dentry**'使其对用户可见。

对象名称冲突（重要）
--------------------------------
POSIX和对象存储是两种不同类型的存储产品，对象存储是一种键-值对存储服务。所以在对象存储中，名称为'*a/b/c*'和名称为'*a/b*'的对象是两个完全没有冲突的对象。

不过ChubaoFS是基于POSIX设计的。根据语义转换规则，对象名'*a/b/c*'中的'*b*'部分转换为文件夹'*a*'下的文件夹'*b*'，对象名'*a/b*'中的'*b*'部分转换为文件夹'*a*'下的文件'*b*'。

类似于上面这样的对象名称在ChubaoFS中是冲突的。

支持的S3兼容接口
----------------------------

桶接口
^^^^^^^^^^^

.. csv-table::
    :header: "API", "Reference"

    "``HeadBucket``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadBucket.html"
    "``GetBucketLocation``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketLocation.html"

对象接口
^^^^^^^^^^^

.. csv-table::
    :header: "API", "Reference"

    "``HeadObject``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html"
    "``PutObject``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html"
    "``GetObject``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html"
    "``ListObjects``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjects.html"
    "``ListObjectsV2``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html"
    "``DeleteObject``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html"
    "``DeleteObjects``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObjects.html"
    "``CopyObject``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html"

并发上传接口
^^^^^^^^^^^^^^^^^^^^^

.. csv-table::
    :header: "API", "Reference"

    "``CreateMultipartUpload``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html"
    "``ListMultipartUploads``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListMultipartUploads.html"
    "``AbortMultipartUpload``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_AbortMultipartUpload.html"
    "``CompleteMultipartUpload``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html"
    "``ListParts``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListParts.html"
    "``UploadPart``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html"
    "``UploadPartCopy``", "https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPartCopy.html"

支持的SDK
--------------
Object Node提供兼容S3的对象存储接口，所以可以直接使用原生的Amazon S3 SDKs来操作文件。

.. csv-table::
   :header: "Name", "Language", "Link"

    "AWS SDK for Java", "``Java``", "https://aws.amazon.com/sdk-for-java/"
    "AWS SDK for JavaScript", "``JavaScript``", "https://aws.amazon.com/sdk-for-browser/"
    "AWS SDK for JavaScript in Node.js", "``JavaScript``", "https://aws.amazon.com/sdk-for-node-js/"
    "AWS SDK for Go", "``Go``", "https://docs.aws.amazon.com/sdk-for-go/"
    "AWS SDK for PHP", "``PHP``", "https://aws.amazon.com/sdk-for-php/"
    "AWS SDK for Ruby", "``Ruby``", "https://aws.amazon.com/sdk-for-ruby/"
    "AWS SDK for .NET", "``.NET``", "https://aws.amazon.com/sdk-for-net/"
    "AWS SDK for C++", "``C++``", "https://aws.amazon.com/sdk-for-cpp/"
    "Boto3", "``Python``", "http://boto.cloudhackers.com"


