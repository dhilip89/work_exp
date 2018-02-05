## 文件操作  

* 查看目录文件  * $ hadoop dfs -ls /user/cl  *  

* 创建文件目录  * $ hadoop dfs -mkdir /user/cl/temp  *  

* 删除文件  * $ hadoop dfs -rm /user/cl/temp/a.txt  *  

* 删除目录与目录下所有文件  * $ hadoop dfs -rmr /user/cl/temp  *  

## 上传文件  

* 上传一个本机/home/cl/local.txt到hdfs中/user/cl/temp目录下  

* $ hadoop dfs -put /home/cl/local.txt /user/cl/temp  *  

## 下载文件  

* 下载hdfs中/user/cl/temp目录下的hdfs.txt文件到本机/home/cl/中  

* $ hadoop dfs -get /user/cl/temp/hdfs.txt /home/cl  *  

* 查看文件  * $ hadoop dfs –cat /home/cl/hdfs.txt  *  

## Job操作

 * 提交MapReduce Job, Hadoop所有的MapReduce Job都是一个jar包

 * $ hadoop jar <local-jar-file> <java-class> <hdfs-input-file> <hdfs-output-dir>

 * $ hadoop jar sandbox-mapred-0.0.20.jar sandbox.mapred.WordCountJob /user/cl/input.dat /user/cl/outputdir  *  * 杀死某个正在运行的Job  * 假设Job_Id为：job_201207121738_0001  * $ hadoop job -kill job_201207121738_0001

 
## map reduce  yarn
> yarn允许应用程序为任务请求任意规模的内存量(需在预定范围内),节点管理器从一个内存池中中分配内存,这意味着可同时运行数据依赖于内存需求量,而非槽数量
>
> 基于槽的模型导致集群未被充分利用,原因是map和reduce的槽之间的比率是固定的,不同的阶段,map-reduce的槽的需求会不断变化. 