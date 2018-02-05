## 数据面试-Kafka
### Kafka相关
1. 容错机制
	* 分区备份，存在主备partition
2. 同一topic不同partition分区
	* ？？？？
3. kafka数据流向
	* Producer  leader partition  follower partition(半数以上) consumer

4. kafka中存储目录data/dir.....topic1和topic2怎么存储的，存储结构，data.....目录下有多少个分区，每个分区的存储格式是什么样的？
	* 1、topic是按照“主题名-分区”存储的
	* 2、分区个数由配置文件决定
	* 3、每个分区下最重要的两个文件是0000000000.log和000000.index，0000000.log以默认1G大小回滚。