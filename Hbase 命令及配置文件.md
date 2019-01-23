# Hbase 命令及配置文件

## Shell 命令

进入hbase 命令模式

```shell
hbase shell
```

版本，状态，用户

```mysql
version
status
whoami
```

查看表

```mysql
list
```

创建表

```
create 'table_name','c1'
```

查看表是否存在

```mysql
exists 'table_name'
```

查看表结构

```mysql
desc 'table_name'
```

修改表结构,将列族‘c1’的生存期限改为30天(2592000s)

```mysql
disable 'table_name'
alter 'table_name',{NAME=>'c1',TTL=>'2592000'}
enable 'table_name'
```

插入数据

```mysql
语法：put <table>,<rowkey>,<family:column>,<value>,<timestamp>

put 'table_name','rowkey001','f1:col1','value1'
```

查询单列的值

```mysql
语法：get <table>,<rowkey>,[<family:column>,....]

get 'table_name','rowkey001', 'f1:col1' 
```

查询所有列的值

```mysql
get 'table_name','rowkey001'
```

扫描前3行数据

```mysql
scan 'table_name',{LIMIT=>3}
```

查询表中的行数，每10行显示一次，缓存为200

```mysql
count 'table_name',{INTERAL=>10,CACHE=>200}
```

删除table_name表中，rowkey001中的f1：col2的数据

```mysql
语法：delete <table>, <rowkey>, <family:column> , <timestamp>,必须指定列名

delete 'table_name','rowkey001','f1:col2' 
```

删除table_name表中rowkey002这行数据

```mysql
deleteall 'table_name','rowkey002'
```

删除table_name表中的所有数据

```mysql
truncate 'table_name' 
```

## 配置文件

hbase-env.sh配置

```shell
#BIGDATA environment set
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/usr/java/jdk1.8.0_171
export HBASE_CLASSPATH=/usr/hadoop/hadoop-2.7.3/etc/Hadoop
```

hbase-site.xml配置

```xml
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://master:9000/hbase</value>
</property>
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<property>
  <name>hbase.master</name>
  <value>hdfs://master:6000</value>
</property>
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>master,slave1,slave2</value>
</property>
<property>
  <name>hbase.zookeeper.property.dataDir</name>
  <value>/usr/zookeeper/zookeeper-3.4.10</value>
</property>
```

regionservers配置

```shell
slave1
slave2
```

配置环境变量

```shell
vim /etc/profile

#set hbase environment
export HBASE_HOME=/usr/hbase/hbase-1.2.4
export PATH=$PATH:$HBASE_HOME/bin

source /etc/profile
```

运行Hbase

```shell
$HBASE_HOME/bin/start-hbase.sh
```

