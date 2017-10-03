#MySql

## 服务器性能剖析

### 使用SHOW GLOBAL STATUS
```
mysqladmin ext -i1 | awk '
/Queries/{q=$4-qp;qp=$4}
/Threads_connected/{tc=$4}
/Threads_running/{printf "%5d %5d %5d\n", p, tc, $4}'
```
此命令计算并输出每秒的查询数，Threads_connected和Threads_running(表示当前正在执行查询的线程数), 这三个数据的趋势对于服务器级别偶尔停顿的敏感性很高。一般发生此类问题时，根据原因的不同和应用连接数据库的方式的不同，每秒的查询数一般会下跌，而其他两个则至少有一个会出现尖刺。如果应用使用了连接池，Threads_connected没有变化，但正在执行查询的线程数明显上升，同时每秒的查询数相比正常数据有严重的下跌。
如何解析这个现象：

* 其中之一是服务器内部碰到了某种瓶颈，导致新查询在开始执行前因为需要获取老查询正在等待的锁而造成堆积。这一类的锁一般也会对应用服务器造成后端压力，使得应用服务器也出现排队问题

* 另外一个原因是服务区突然遇到了大量查询请求的冲击，比如前端的memcached突然失效导致的查询风暴。 

#### 使用 SHOW PROCESSLIST
```
mysql -e 'SHOW PROCESSLIST\G' | grep State: | sort | uniq -c | sort -rn
```
可以看到线程的状态有:freeing items, end, cleaning up, logging slow query，大量的线程处于'freeing items'状态是出现了大量有问题查询的很明显的特征和指示

如果MySql服务器的版本较新，也可以直接查询INFOMATION_SCHEMA中的PROCESSLIST表；或者使用innotop工具以较高频率刷新。

* InnoDB内部的争用和脏块刷新，会导致线程堆积。
* 一个经典的例子是很多查询处于"Locked"状态，这是MyISAM的一个经典问题，它的表级锁，在写请求较多时，可能迅速导致服务器级别的线程堆积。