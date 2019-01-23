# Hive 命令及相关配置文件

## Shell命令

### Hive数据库的操作

创建db数据库

```mysql
create database db;
```

查看所有数据库

```mysql
show databases;
```

查看数据库的路径及信息

```mysql
describe database db;
```

删除名为db的数据库

```mysql
drop database db;
```

使用数据库db

```mysql
use db;
```

### Hive数据表的操作

创建一个名为cat的内部表

```mysql
create table cat(cat_id string,cat_name string) row format delimited fields terminated by '\t'  stored as textfile;
```

创建一个名为cat1的外部表

```mysql
create external table cat1(cat_id string,cat_name string);
```

查看所有的表

```mysql
show tables;
```

修改cat表结构，增加两个字段

```mysql
alter table cat add columns (group_id string,cat_code string);
```

修改表名cat1为cat2

```mysql
alter table cat1 rename to cat2;
```

查看cat表结构

```mysql
desc cat;
```

创建与cat结构相同的表cat3

```mysql
create table cat3 like cat;
```

### 数据的导入和导出

将Linux本地文件导入到Hive的cat表中

```mysql
load data local inpath '/data/ab' into table cat;
```

将HDFS中/myhive/ab1的内容导入到Hive的cat表中

```mysql
load data inpath '/myhive/ab1' into table cat;
```

从表cat中导入数据到cat2中

```mysql
insert into table cat2 select * from cat;
```

or

```mysql
insert overwrite table cat2 select * from cat;
```

查询表cat中的数据

```mysql
select * from cat limit 10;
```

将表cat中数据导出到Linux本地文件‘/data/out’

```mysql
insert overwrite local directory '/data/out' select group_id,concat('\t',group_name) from cat;
```

将表cat中数据导出到HDFS中

```mysql
insert overwrite directory '/myhive2/out' select group_id,concat('\t',group_name) from cat;
```

创建表cat3并从cat中导入数据

```mysql
create table cat3 as select * from cat;
```

### Hive 分区表的操作

创建分区表goods

```mysql
create table goods(goods_id string,goods_status string) partitioned by (cat_id string) row format delimited fields terminated by '\t'; 
```

向分区表中导入数据

1、若不在hive上，需先将数据上传的hive上

```mysql
create table goods_1(goods_id string,goods_status string,cat_id string) row format delimited fields terminated by '\t';
```

2、若数据在hive上，可直接导入goods中

```mysql
insert into table goods partition(cat_id = ‘52052’) select goods_id,goods_status from goods_1 where cat_id='52052';
```

查看表goods中的分区

```mysql
show partitions goods;
```

修改表分区

```mysql
alter table goods partitions(cat_id=52052) rename to partition(cat_id=52051);
```

删除goods中的表分区

```mysql
alter table goods drop if exists partition (cat_id='52051');
```

### Hive 查询

普通查询

```mysql
select * from table buyer_log limit 10;
```

别名查询

```mysql
select b.id,b.ip from buyer_log b limit 10;
```

限定查询(where)

```mysql
select buyer_id from buyer_log where opt_type=1 limit 10;
```

两表或多表联合查询

```mysql
select l.dt,f.goods_id from buyer_log l,buyer_favorite f where l.buyer_id=f.buyer_id limit 10;
```

多表插入

```mysql
from buyer_log
insert overwrite table buyer_log1 select *
insert overwrite table buyer_log2 select *;
```

多目录输出文件

```mysql
from buyer_log
insert overwrite local directory '/data/hive3/out' select *
insert overwrite local directory '/data/hive3/out1' select *;
```

### Hive分组排序

全局排序order by

```mysql
select * from table goods order by click_num desc limit 10;
```

局部排序sort by

```mysql
select * from goods sort by goods_id; 
```

分组查询group by

```mysql
select dt,count(buyer_id) from goods group by dt;
```

分发distribute by(数据按buyer_id分发到多个文件里)

```mysql
insert overwrite local directory '/data/out' select * from goods distribute by buyer_id;
```

####Order by 与Sort by 对比

Order by的查询结果是全部数据全局排序，它的Reduce数只有一个，Reduce任务繁重，因此数据量大的情况下将会消耗很长时间去执行，而且可能不会出结果，因此必须指定输出条数。

Sort by是在每个Reduce端做排序，它的Reduce数可以有多个，它保证了每个Reduce出来的数据是有序的，但多个Reduce出来的数据合在一起未必是有序的，因此在Sort by做完局部排序后，还要再做一次全局排序，相当于先在小组内排序，然后只要将各小组排序即可，在数据量大的情况下，可以提升不少的效率。

####Distribute by 与Group by 对比

Distribute by是通过设置的条件在Map端拆分数据给Reduce端的，按照指定的字段对数据划分到不同的输出Reduce文件中。

Group by它的作用是通过一定的规则将一个数据集划分成若干个小的区域，然后针对若干个小区域进行数据处理，例如某电商想统计一年内商品销售情况，可以使用Group by将一年的数据按月划分，然后统计出每个月热销商品的前十名。

两者相比，都是按Key值划分数据，都使用Reduce操作，唯一不同的是Distribute by只是单纯的分散数据，而Group by把相同Key的数据聚集到一起，后续必须是聚合操作。

## 配置文件

配置环境变量

```shell
vim /etc/profile

#set hive environment
export HIVE_HOME=/usr/hive/apache-hive-2.1.1-bin
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile
```

hive-env.sh配置

```shell
HADOOP_HOME=/usr/hadoop/hadoop-2.7.3
```

hive-site.xml(slave1)

```xml
<configuration>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive_remote/warehouse</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://slave2:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>123456</value>
</property>
<property>
  <name>hive.metastore.schema.verification</name>
  <value>false</value>
</property>
<property>
  <name>datanucleus.schema.autoCreateAll</name>
  <value>true</value>
</property>
</configuration>
```

hive-site.xml(master)

```xml
<configuration>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive_remote/warehouse</value>
</property>
<property>
  <name>hive.metastore.local</name>
  <value>false</value>
</property>
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://slave1:9083</value>
</property>
</configuration>
```

启动hive server（slave1)

```shell
$HIVE_HOME/bin/hive --service metastore
```

启动hive client（master)

```shell
$HIVE_HOME/bin/hive
```

### Shell 脚本

```shell
vim sh1
cd /apps/hive/bin;
hive -e 'show databases;'

chmod +x sh1
./sh1
```

