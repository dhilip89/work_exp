## 数据面试-Storm
### Storm相关
1. 开发流程，容错机制
	* 开发流程：
		* 1、写主类（设计spout和bolt的分发机制）
		* 2、写spout收集数据
		* 3、写bolt处理数据，根据数据量和业务的复杂程度，设计并行度。
	* 容错机制：采用ack和fail进行容错，失败的数据重新发送。

2. 唯storm 如果碰上了复杂逻辑,需要算很长的时间,你怎么去优化?
	* 拆分复杂的业务到多个bolt中，这样可以利用bolt的tree将速度提升

3. 公司技术选型可能利用storm 进行实时计算,讲解一下storm 