# Hive编程指南
## 每二章 基础操作
### Hive 是什么
> 所有的hive客户端都需要一个metastoreservice(无数据服务)


### 2.6Hive命令
### 2.7Hive命令行界面

#### hive中的变量和属性命名军空间

###
命名空间     | 使用权限 | 描述
-------- | --- |---
hivevar| r/w |用户自定
hiveconf    | r/w | hive相关的配置属性
system     | r/w |Java定义的配置属性
env | w | shell环境定义的环境变量

```
hive --hiveconf 可以指定当会命名空间

//set
hive> set hivevar:foo=bar2;
//get
hive> set hivevar:foo

//一次使用命令 加上 -S 可以去掉ok, time taken 等字符, 然后重定向输入文件
hive -e "selct * from table limit 3";

//查找历史命令
hive -S -e "set" | grep warehouse

//从文件中执行hive查询
hive -f /path/to/file/hive.sql

//源数据
create table src(s string);
echo "one row" > /tmp/myfile
hive -e "load data local inpath '/tmp/myfile' into table src";

//可以把频繁执行的命令放入hiverc文件
//在$HOME/.hiverc写入以下内容:
ADD JAR /path/to/custom_live.extensions.jar;
set useSSL=true;
set hive.cli.print.current.db=true;//修改cli提示符为当前所在数据库
set hive.exec.model.local.auto=true;//鼓励使用本地模式

//tab可以自动补全

//执行shell命令
hive> ! /bin/echo "what up dog";
//hive中使用hodoop的dfs命令,只需要去掉hadoop就可以
hive>dfs -ls /;

//hive注释以--开头.如:
--This is best hive script.
select * from ...

//显示字段名称
hive> set hive.cli.print.header=true;

```

## 第三章 数据类型和文件格式
### 基本数据类型
数据类型 | 长度 | 例子
-----| --- | ---
TINYINT ||
SAMLLINT||
INT||
BIGINT||
BOOLEAN||
FLOAT||
DOUBLE||
STRING||
TIMESTAMP(v0.8.0+)|整数,浮点数或字符串|
BINARY(v0.8.0+)|字节数组|

### 集合数据类型
数据类型 | 描述 | 字面语法示例
---- | --- | ---
STRUCT |结构体对象,可以通过点访问元素内容 |struct('John', 'Doe')
MAP |键-值对元组集合| map()
ARRAY |相同类型和名称的变量的集合|

```
create table employees(
	name string,
	salary float,
	suborinates array(string),
	deductions map(string, float),
	address struct<street:string, city:string, state:string, zip:int>
);
```
hive有默认的记录和字段分割符

分隔符 | 描述
----| ---
\n | 换持符
^A(Ctrl+A)|用于分割字段(列),在create table时可以用八进制编码\001表示
^B | 用于分隔array或struct中的元素,或于map中键-值对之间分隔,在create table中可以用八进制\002表示
^C | 用于map中键-值之间的分隔,在create table语句中可以以八进制\003表示.

```
create table employees(
	name string,
	salary float,
	suborinates array(string),
	deductions map(string, float),
	address struct<street:string, city:string, state:string, zip:int>
)
row format delimited
fieleds terminated by '\001',
collection items terminated by '\002',
map keys terminated by '\003',
lines terminated by '\n',
stored as textfile;
```

下表数据按逗号进行分割:
```
create tabel some_data(
	first float,
	second float,
	third float,
)
row format delimited
fields terminated by ',';
```

### 读时模式
hive不会在数据加载时进行验证,而是在查询时进行,所以是读时模式(schema on read)

如果不每行记录中的字段个数少于对应的模式中定义的字段的个数,那么查询的结果有很多的null值,如果某些字段是数值型的,但hive在读取时发现存储在非数据值型的字符串时,那些字段将返回null值.

## 第4章 HiveQL:数据定义
hive 不支持行级插入操作,更新操作和删除操作,也不支持事务.

### Hive中的数据库
hive中的数据库本质是仅是表的一个目录或是命名空间.
```
hive> create database if not exists financials;

//用正则表达式筛选数据
hive>show databases like 'h.*';

//手动指定位置
hive>create database financials location '/path/to/data'

//添加注释
hive>cteate database financials comment 'holds all financial tables';

//查看库信息
hive>describe database financials;

//use
hive>user financials;

//删除数据库
hive>drop database if exists financials;

//删除有表的数据库
hive>drop database if exists financial cascade;
/如果是restrict,cascade也不可使,必须删除所有表才能删了库

//修改数据库
hive>alter database financials set dbproperties('edited-by'='Joe Dba');

//创建表
create table if not exists mydb.employees(
	name string comment 'employee name',
	salary float commnet 'employee salary',
	subordinates array<string> commnet 'names of subordinates',
	deductions mpa<string, float> comment 'keys ar deductions names, values are precentages',
	address struct<street:string, city:string, state:string, zip:int> comment 'home address'
) comment 'description of the table'
tblproperties ('creator'='me', 'created_at'='2013-12-12')
location '/user/hive/warehouse/mydb.db/employees'

//hive会自动增加两个属性:last_modified_by, last_moditfied_time;

//拷贝一张表的表模式,不带数据
create table if not exist mydb.employee2 like mydb.employees;

//查看表详细信息
hive>describe extended mydb.employees;
hive>describe mydb.employees.salary;

//external,这个表是外部的,删除该表并不会外部数据,不过表的元数据会被删除
create external table if not exists stocks(
	exchange string,
	...
)
row formated fields terminated by ',',
location '/data/stocks';

//查看表是否是外部表
hive>describe extended tablename;

//数据分区
create table employees(
name string,
salary float,
suborinates array<string>,
deductions map<string, float>,
address struct<street:string, city:string, state:string, zip:int>) partitioned by (country string, state string);


//查看表的分区
hive>show partitions employees;
hive>show partitions employees partition(country='US');
hive>describe extended employees;
hive>describe extended log_messages partition(year=2012, month=1, day=2);

//load 数据
load data local inpath '/path/to/file' into table employees partition(country='US', stata='CA');


//外部分区表
create external table if not exists log_messages(
hms int,
severity string,
server string,
process_id int,
message string
)
partitioned by (year, int, month int, day int)
row format delimited fields terminated by '\t';

//修改表,修改分区
alter table log_messages add partition(year=2012, month=1, day=2) location 'hdfs://......';

//自定义表的存储格式
SEQUENCEFILE, RCFILE, 这两种格式都是有二进制编码和压缩(可选)

create external table if not exists stocks(
exchage string,
symbol string,
ymd string,
price_open float,
price_high float,
price_how float,
price_close float,
volume int,
price_adj_close float
)
clustered by (exchange, symbol)
sorted by (ymd asc)
insert 96 buckets
row format delimited fields terminated by ','
location '/data/stocks';


//表重命名
alter table log_messages rename to logmsgs;

//增加,修改,删除表分区
alter table log_messages add if not exists
partition (year=2011, month=1, day=1) location '/logs/2011/01/01',
.....;
alter table log_messages drop if not exists
partition (year=2011, month=1, day=1) location '/logs/2011/01/01'

//修改列信息
alter table log_messages
change column hms hours_minutes_seconds int
comment 'the hours, minutes, and seconds part of the timestamp'
after serverity;

//增加列
alter table log_messages add columns(
app_name string comment 'application name',
session_id long comment 'the current id'
);

//删除或替换列
alter table log_messages replace columns(
hours_mins_secs int comment 'hour, minutes, ...',
serverity string comment '...',
message string comment '...'
);

//修改存储属性
alter table log_messages
partition(year=2013, month=1, day=1)
set fileformat sequencefile;

alter table using_json_storge
set serde 'com.example.JSONSerDe'
with serdeproperties(
'prop1'='value1',
'prop2'='value2'
);

//防止被删除 enable disable
alter table log_messages partition(year=1012, month=1, day=1) enable NO_DROP;
//防止被查询
alter table log_messages partition(year=2012, month=1,day=1) enable OFF_LINE;
```

## 第5章 HiveQL:数据操作
```
//向管理表中装载数据 overwrite会删除目标文件夹中之前存在的数据
//inpath中不可能包含文件夹
load data local inpath '/home/work/src/programming_hive/data/employees'
overwrite into table employees
partition(country='US', state='CA');

//插入数据
from staged_employees se
insert overwrite table employees
	partition(contry='US', state='OR')
		select * where se.cnty = 'US' and se.st='OE'
....;

//动态分区
insert overwrite table employees
partition(country, state)
select ..., se.cnty, se.st from staged_employees se;

//单个查询语句中创建表并加载数据
create table ca_employees
as select name, salary address
from employees se where se.state='CA';

//导出数据
hadoop fd -cp source_path target_path
或者
insert overwrite local DIRECTORY '/tmp/employees'
select name, salary, address
from employees se
where se.state = 'CA';

//导出数据至多个文件目录
from staged_employees se
insert overwrite directory '/tmp/or_employees'
	select * where se.city = 'US' and se.state='OR'
insert overwrite directory '/tmp/ca_employees'
	select * from se.city = 'US' and se.state='CA'
....;
```

## 第6章 HiveQL:查询
```
select name, salary from employees;
select name, suborinates from employees;
select name, deductions from employees;
select name, address from employees;

//引用集合元素
select name, suborinates[0] from employees;

//引用map元素
select name, deductions['State Taxes'] from employees;

//引用struct元素
select name, address.city from employees;

//使用正则表达
select symbol, `price.*` from stocks;

//使用列值进行计算
select upper(name), salary, deductions['Federal Taxes'],
round(salary * (1-deductions['Federal Taxes'])) from employees;


```

算术运算符

当进行算术运算时,注意数据溢出或下溢,乘法和除法会引发这个问题

数学函数

返回值类型 | 样式 | 描述
---|---|---
BIGINT | round(DOUBLE D)| --
DOUBLE | round(DOUBLE d, INT n) |--
BIGINT | floor(DOUBLE d) |
BIGINT | ceil(DOUBLE d), ceiling(DOUBLE d)
DOUBLE | rand(), rand(INT, seed) |
DOUBLE | exp(DOUBLE d) |
DOUBLE | ln(DOUBLE d)|
DOUBLE | log10(DOUBLE d) |
DOUBLE | log2(DOUBLE d)|
DOUBLE | log(DOUBLE base, DOUBLE d)|
DOUBLE | pow(DOUBLE d, DOUBLE p) |
DOUBLE | sqrt(DOUBLE d)
STRING | bin(DOUBLE i) |
STRING | hex(DOUBLE i) |
STRING | hex(STRING i) |
STRING | unhex(STRING i)|
STRING | conv(BIGINT num, INT from_base, INT to_base)| 
STRING | conv(STRING num, INT from_bae, INT to_base) |
DOUBLE | ads(DOUBLE d) |
INT | pmod(INT i1, INT i2) |
DOUBLE | pmod(DOUBLE d1, DOUBLE d2) |
DOUBLE | sin(DOUBLE d)|
DOUBLE | asin(DOUBLE d) |
DOUBLE | cos(DOUBLE d) |
DOUBLE | acos(DOUBLE d) |
 |tan(DOUBLE d)|
 |atan(DOUBLE d)|
 |degree(DOUBLE d)|
 |radians(DOUBLE d)|
 |positive(INT i)|
 |positive(DOUBLE d)|
 |negative(INT i)|
 |negative(DOUBLE d)|
 |sign(DOUBLE d)|
 |e()|
 |pi()|

聚合函数

```
select count(*), avg(salary) from employees;
```

返回值类型|样式|描述
---| --- | ----
 |count(*) |
 |count(expr) |
 |count(distict expr[, expr_.]) |
 |sum(col) |
 |sum(distict col) |
 |avg(col) |
 |avg(distict col) |
 |min(col) |
 |max(col) |
 |varirance(col),var_pop(col) |
 |var_samp(col) |
 |stddev_pop(col) |
 |stddev_samp(col) |
 |covar_pop(col1, col2) |
 |covar_samp(col1, col2) |
 |corr(col1, col2) |
 |percentile(bigint int_expr, p) |
 |percentile(bigint int_expr, array(p1[, p2])) |
 |percentile_approx(double col, p[, nb]) |
 |percentile_approx(double col, array(p1[,p2])[,nb]) |
 |histogram_numeric(col, nb) |
 |collect_set(col) |
 
 ```
 set hive.map.aggr=true;
 select count(*), avg(salary) from table;
 
 select count(distict symbol) from table;
 
 //将一个字段转换成多个字段
 select explode(suborinates) as sub from employees;
 
 //解析url
 select parse_url_tuple(url, 'HOST', 'PATH', 'QUERY') 
 as ('host', 'path', 'query') from url_table;
 
 ```
 
 表生成函数
 
 返回值类型 | 样式 | 描述
 ---|---|---
 N行结果|explode(array d)|返回0到多行结果,每行对应输入的array数组中的一个元素
 N行结果|explode(map d)|返回0到多行结果,每行对应每个map键值对,一个字段是map键,一个是map的值
 数组的类型 | explode(array<type> a) |对于a中的每个元素,explode()会生成一行记录包含这个元素
结果插入表中| inline(array<struct[, strcut]>) | 将结构体数组插入到表中
 tuple | json_tuple(struct jsonStr, p1, p2,...) | 本函数接收多个标签名称,对输入的json字符串进行处理,这个get_json_object这个udf类似
tuple |  parse_url_tuple(url, partname1, partname2....partnamen) n>=1 |从url中解析出n个部分信息
n个结果 | stack(int ,col1, ..., colm) | 把m个列转换成n行,每行有m/n个字段,n必须是常数

其他内置函数

返回值类型|样式|描述
---|---|---
--|--|--

```
//limit 语句
select upper(name), salary, deductions['Federal Taxes'],
round(salary * (1-deductions['Federal Taxes'])) from employees limit 2;

set hive.cli.print.header=true;  // 打印列名 
set hive.cli.print.row.to.vertical=true;   // 开启行转列功能, 前提必须开启打印列名功能 
set hive.cli.print.row.to.vertical.num=1; // 设置每行显示的列数 
set hive.exec.mode.local.auto=true;//本地模式


//列别名
select upper(name) as name, salary, deductions['Federal Taxes'] as fed_taxes,
round(salary * (1-deductions['Federal Taxes'])) as salary_minus_fed_taxes from employees;

//嵌套select语句
from(
select upper(name) as name, salary, deductions['Federal Taxes'] as fed_taxes, round(salary * (1-deductions['Federal Taxes'])) as salary_minus_fed_taxes from employees
) as e
select e.name, e. salary_minus_fed_taxes where e. salary_minus_fed_taxes > 70000;

//case...where...then
select name, salary,
case
when salary < 50000.0 then 'low'
when salary >= 50000.0 and salary < 70000.0 then 'middle'
when salary >= 70000.0 and salary < 100000.0 then 'high'
else 'very hight'
end as bracket from employees;
```

where 语句
```
select upper(name) as name, salary, deductions['Federal Taxes'] as fed_taxes from employees where round(salary * (1-deductions['Federal Taxes'])) >= 70000.0;

//不能在where语句中使用别名
//不过可以使用嵌套
select e.* from
(select name, salary, deductions['Federal Taxes'] as ded, salary*(1-deductions['Federal Taxes']) as salary_minus_fed_taxes from employees) as e
where round(e.salary_minus_fed_taxes) > 70000;
```

谓词操作符

操作符|支持数据类型|描述
---|---|---
a = b||
a<=>b||
a==b||错
a<>b, a!=b||
a<b||
a<=b||
a>b||
a>=b||
a [not] between a and c||
a is NULL||
a is not NULL||
a [not] like b||
a rlike a||
a rlike b, regexp b||

```
浮点数比较,出现0.2>0.2
解决的办法有:
	1.修改数据类型
	2.cast转换数据类型
	
select name, salary, deductions['Federal Taxes'] from employees where deductions['Federal Taxes'] > cast(0.2 as float);


//like和rlike
select name, address.street from employees where address.street like '%Ave.';

select name, address.city from employees where address.city like 'O%';

//rlike
select name, address.street from employees where address.street rlike '.*(Chicago|Ontario).*';


//group by
select year(ymd), avg(price_close) from stocks
where exchange = 'NASDAQ' and symbol='APPL'
group by year(ymd);

//having
select year(ymd), avg(price_close) from stocks
where exchange = 'NASDAQ' and symbol='AAPL'
group by year(umd) having avg(proce_close) > 50.0;

要不然就要嵌套
select s2.year, s2.avg from 
(select year(ymd) as year, avg(price_close) as avg from stocks where exchange='NASDAQ' and symbol='AAPL' group by year(ymd)) s2 where s2.avg > 50.0;

//join join只支持等值操作
select a.ymd, a.price_close, b.price_close from stocks a join stocks b on a.ymd = b.ymd where a.symbol = 'AAPL' and b.symbol = 'IBM';

```

