Dentry调试命令
=======================

获取Dentry信息
---------------

.. code-block:: bash

   curl -v 'http://127.0.0.1:9092/getDentry?pid=100&name="aa.txt"&parentIno=1024'


.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "pid", "integer", "meta partition id" 
   "name", "string", "directory or file name"
   "parentIno", "integer", "parent directory inode id"
    
获取指定目录下全部文件
-------------------------------

.. code-block:: bash

   curl -v 'http://127.0.0.1:9092/getDirectory?pid=100&parentIno=1024'

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "pid", "integer", "partition id"
   "ino", "integer", "inode id" 

获取指定分片的全部目录信息
----------------------------------

.. code-block:: bash

   curl -v 'http://127.0.0.1:9092/getAllDentry?pid=100'

.. csv-table:: Parameters
   :header: "Parameter", "Type", "Description"
   
   "pid", "integer", "partition id"