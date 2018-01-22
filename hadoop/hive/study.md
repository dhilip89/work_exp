## hive动态分区
### 一）hive中支持两种类型的分区：
* 静态分区SP（static partition）
* 动态分区DP（dynamic partition）

静态分区与动态分区的主要区别在于静态分区是手动指定，而动态分区是通过数据来进行判断。详细来说，静态分区的列实在编译时期，通过用户传递来决定的；动态分区只有在SQL执行时才能决定。

### 二）实战演示如何在hive中使用动态分区

1、创建一张分区表，包含两个分区dt和ht表示日期和小时

```
CREATE TABLE partition_table001   
(  
    name STRING,  
    ip STRING  
)  
PARTITIONED BY (dt STRING, ht STRING)  
ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t";  
```

2、启用hive动态分区，只需要在hive会话中设置两个参数：

```
set hive.exec.dynamic.partition=true;  
set hive.exec.dynamic.partition.mode=nonstrict;
```
3、把partition_table001表某个日期分区下的数据load到目标表partition_table002
使用静态分区时，必须指定分区的值，如：

```
create table if not exists
partition_table002 like partition_table001;

insert overwrite table partition_table002
partition (dt='20150617', ht='00') select name, ip from partition_table001 where dt='20150617' and ht='00';
```

4、使用动态分区

```
insert overwrite table partition_table002 
partition (dt, ht) select * from 
partition_table001 where dt='20150617';
```
hive先获取select的最后两个位置的dt和ht参数值，然后将这两个值填写到insert语句partition中的两个dt和ht变量中，即动态分区是通过位置来对应分区值的。原始表select出来的值和输出partition的值的关系仅仅是通过位置来确定的，和名字并没有关系，比如这里dt和st的名称完全没有关系。
只需要一句SQL即可把20150617下的24个ht分区插到了新表中。

### 三）静态分区和动态分区可以混合使用
1、全部DP

```
INSERT OVERWRITE TABLE T PARTITION (ds, hr) 
SELECT key, value, ds, hr FROM srcpart 
WHERE ds is not null and hr>10;  
```
2、DP/SP结合

```
INSERT OVERWRITE TABLE T PARTITION (ds='2010-03-03', hr)  
SELECT key, value, /*ds,*/ hr FROM srcpart
WHERE ds is not null and hr>10; 
``` 

3、当SP是DP的子分区时，以下DML会报错，因为分区顺序决定了HDFS中目录的继承关系，这点是无法改变的

```
-- throw an exception  
INSERT OVERWRITE TABLE T PARTITION (ds, hr = 11)  
SELECT key, value, ds/*, hr*/ FROM srcpart
WHERE ds is not null and hr=11;  
```

4、多张表插入

```
FROM S  
INSERT OVERWRITE TABLE T PARTITION (ds='2010-03-03', hr)  
SELECT key, value, ds, hr FROM srcpart 
WHERE ds is not null and hr>10  
INSERT OVERWRITE TABLE R PARTITION 
(ds='2010-03-03, hr=12)  
SELECT key, value, ds, hr from srcpart 
where ds is not null and hr = 12;  
```

5、CTAS，（CREATE-AS语句），DP与SP下的CTAS语法稍有不同，因为目标表的schema无法完全的从select语句传递过去。这时需要在create语句中指定partition列

```
CREATE TABLE T (key int, value string) PARTITIONED BY (ds string, hr int) AS  
SELECT key, value, ds, hr+1 hr1 FROM srcpart WHERE ds is not null and hr>10; 
```
 
6、上面展示了DP下的CTAS用法，如果希望在partition列上加一些自己的常量，可以这样做

```
CREATE TABLE T (key int, value string) PARTITIONED BY (ds string, hr int) AS  
SELECT key, value, "2010-03-03", hr+1 hr1 FROM srcpart WHERE ds is not null and hr>10;  
```

四）小结：
通过上面的案例，我们能够发现使用hive中的动态分区特性的最佳实践：对于那些存在很大数量的二级分区的表，使用动态分区可以非常智能的加载表，而在动静结合使用时需要注意静态分区值必须在动态分区值的前面