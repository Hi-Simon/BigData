# 大数据集群VM搭建步骤

## Shell 脚本

```shell
vim sh1
cd /usr
ls


chmod +x sh1
./sh1
```

##基本环境配置

配置网络和主机名**(master,slave1,slave2)**

```shell
vi /etc/hosts

NETWORKING=yes
HOSTNAME=master

hostnamectl set-hostname master
cd /etc/sysconfig/network-scripts/
ls

vi ifcfg-ens33

#set ONBOOT=yes

reboot -f
```

设置主机与地址的映射**(master,slave1,slave2**)

```shell
ip addr    #查看ip地址

vim /etc/hosts

192.168.61.134 master master.root
192.168.61.135 slave1 slave1.root
182.168,61,136 slave2 slave2.root
```

下载相关软件**(master,slave1,slave2**)

```shell
yum install -y vim
yum install -y net-tools
yum install -y wget
yum install -y ntp
```

关闭防火墙**(master,slave1,slave2)**

```shell
systemctl stop firewalld

systemctl status firewalld

vim /etc/profile

#set java environment
export JAVA_HOME=/usr/java/jdk1.8.0_171
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin
#set zookeeper environment
export ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.10
export PATH=$PATH:$ZOOKEEPER_HOME/bin
#set hadoop environment
export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3
export CLASS_PATH=$HADOOP_HOME:$HADOOP_HOME/lib
export PATH=$PATH:$HADOOP_HOME/bin
#set hbase environment
export HBASE_HOME=/usr/hbase/hbase-1.2.4
export PATH=$PATH:$HBASE_HOME/bin
#set hive environment
export HIVE_HOME=/usr/hive/apache-hive-2.1.1-bin
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile
```

时间同步**(master)**

```shell
tzselect

5
9
1
1

vim /etc/ntp.conf

server 127.127.1.0
fudge 127.127.1.0 stratum 10

systemctl restart ntpd.service
```

其他机器同步**(slave1,slave2)**

```shell
ntpdate master
```

## 配置ssh免密

产生私密钥**(master,slave1,slave2)**

```shell
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```

配置公钥**(master)**

```shell
cd ~/.ssh/
cat id_dsa.pub >> authorized_keys
ssh master

yes
```

主节点免密登陆子节点**(slave1,slave2)**

```shell
cd ~/.ssh
scp master:~/.ssh/id_dsa.pub ./master_das.pub

yes

cat master_das.pub >> authorized_keys
```

测试**(master)**

```shell
ssh slave1

ssh slave2
```

## 安装JAVA jdk

安装并配置环境**(master)**

```shell
mkdir -p /usr/java
tar -zxvf /opt/soft/jdk-8u171-linux-x64.tar.gz -C /usr/java

```

分发java**(master)**

```shell
scp -r /usr/java root@slave1:/usr/
scp -r /usr/java root@slave2:/usr/
```

## 安装Zookeeper

安装并配置zookeeper**(master)**

```shell
mkdir -p /usr/zookeeper
tar -zxvf /opt/soft/zookeeper-3.4.10.tar.gz -C /usr/zookeeper
```

配置conf/zoo.cfg**(master)**

```shell
cd /usr/zookeeper/zookeeper-3.4.10/conf
scp zoo_sample.cfg zoo.cfg
vim zoo.cfg

dataDir=/usr/zookeeper/zookeeper-3.4.10/zkdata
dataLogDir=/usr/zookeeper/zookeeper-3.4.10/zkdatalog
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888

cd ..
mkdir zkdata
mkdir zkdatalog
cd zkdata
vim myid

1
```

分发到其他主机**(master)**

```shell
scp -r /usr/zookeeper root@slave1:/usr/
scp -r /usr/zookeeper root@slave2:/usr/
```

设置myid**(slave1)**

```shell
vim /usr/zookeeper/zookeeper-3.4.10/zkdata/myid

2
```

设置myid**(slave2)**

```shell
vim /usr/zookeeper/zookeeper-3.4.10/zkdata/myid

3
```

配置环境变量**(master,slave1,slave2)**

```shell
vim /etc/profile

#set zookeeper environment
export ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.10
export PATH=$PATH:$ZOOKEEPER_HOME/bin

source /etc/profile
```

启动zookeeper集群**(master,slave1,slave2)**

```shell
$ZOOKEEPER_HOME/bin/zkServer.sh start
```

查看zookeeper状态**(master,slave1,slave2)**

```shell
$ZOOKEEPER_HOME/bin/zkServer.sh status
```

## 安装Hadoop

安装并配置Hadoop**(master)**

```shell
mkdir -p /usr/hadoop
tar -zxvf /opt/soft/hadoop-2.7.3.tar.gz -C /usr/hadoop
```

编辑hadoop-env.sh文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/hadoop-env.sh

export JAVA_HOME=/usr/java/jdk1.8.0_171
```

编辑core-site.xml文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/core-site.xml

<property>
  <name>fs.default.name</name>
  <value>hdfs://master:9000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/usr/hadoop/hadoop-2.7.3/hdfs/tmp</value>
</property>
<property>
  <name>io.file.buffer.size</name>
  <value>131072</value>
</property>
<property>
  <name>fs.checkpoint.period</name>
  <value>60</value>
</property>
<property>
  <name>fs.checkpoint.size</name>
  <value>67108864</value>
</property>
```

编辑yarn-site.xml文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/yarn-site.xml

<property>
  <name>yarn.resourcemanager.address</name>
  <value>master:18040</value>
</property>
<property>
  <name>yarn.resourcemanager.scheduler.address</name>
  <value>master:18030</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address</name>
  <value>master:18088</value>
</property>
<property>
  <name>yarn.resourcemanager.resource-tracker.address</name>
  <value>master:18025</value>
</property>
<property>
  <name>yarn.resourcemanager.admin.address</name>
  <value>master:18141</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

编辑slaves文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/slaves

slave1
slave2
```

编辑master文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/master

master
```

编辑hdfs-site.xml文件**(master)**

```shell
vim /usr/hadoop/hadoop-2.7.3/etc/hadoop/hdfs-site.xml

<property>
  <name>dfs.replication</name>
  <value>2</value>
</property>
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/name</value>
  <final>true</final>
</property>
<property>
  <name>dfs.datanode.data.dir</name>
  <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/data</value>
  <final>true</final>
</property>
<property>
  <name>dfs.namenode.secondary.http-address</name>
  <value>master:9001</value>
</property>
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
<property>
  <name>dfs.permissions</name>
  <value>false</value>
</property>
```

编辑mapred-site.xml文件**(master)**

```shell
cd /usr/hadoop/hadoop-2.7.3/etc/hadoop/
cp mapred-site.xml.template mapred-site.xml
vim mapred-site.xml

<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```

分发hadoop**(master)**

```shell
scp -r /usr/hadoop root@slave1:/usr/
scp -r /usr/hadoop root@slave2:/usr/
```

配置环境变量**(master,slave1,slave2)**

```shell
vim /etc/profile

#set hadoop environment
export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3
export CLASS_PATH=$HADOOP_HOME:$HADOOP_HOME/lib
export PATH=$PATH:$HADOOP_HOME/bin

source /etc/profile
```

格式化hadoop**(master)**

```shell
hadoop namenode -format
```

启动hadoop集群**(master)**

```shell
$HADOOP_HOME/sbin/start-all.sh

jps(master)
namenode
secondarNamenode
ResourceManager

jps(slave)
DataNode
NodeManager
```

## Hbase 安装

安装并配置hbase**(master)**

```shell
mkdir -p /usr/hbase
tar -zxvf /opt/soft/hbase-1.2.4-bin.tar.gz -C /usr/hbase
```

编辑conf/hbase-env.sh文件**(master)**

```shell
vim /usr/hbase/hbase-1.2.4/conf/hbase-env.sh

#BIGDATA environment set
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/usr/java/jdk1.8.0_171
export HBASE_CLASSPATH=/usr/hadoop/hadoop-2.7.3/etc/Hadoop
```

编辑conf/hbase-site.xml文件**(master)**

```shell
vim /usr/hbase/hbase-1.2.4/conf/hbase-site.xml

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

编辑conf/regionservers文件**(master)**

```shell
vim /usr/hbase/hbase-1.2.4/conf/regionservers

slave1
slave2
```

hadoop配置文件导入hbase的conf下**(master)**

```shell
cp /usr/hadoop/hadoop-2.7.3/etc/hadoop/hdfs-site.xml .
cp /usr/hadoop/hadoop-2.7.3/etc/hadoop/core-site.xml .
```

分发hbase**(master)**

```shell
scp -r /usr/hbase root@slave1:/usr/
scp -r /usr/hbase root@slave2:/usr/
```

配置环境变量**(master,slave1,slave2)**

```shell
vim /etc/profile

#set hbase environment
export HBASE_HOME=/usr/hbase/hbase-1.2.4
export PATH=$PATH:$HBASE_HOME/bin

source /etc/profile
```

运行hbase(保证hadoop和zookeeper已开启)**(master)**

```shell
$HBASE_HOME/bin/start-hbase.sh

jps(master)
HMaster

jps(slave)
HRegionServer
```

## 安装Mysql server

安装epel源**(slave2)**

```shell
yum -y install epel-release
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
yum -y install mysql-community-server
systemctl daemon-reload
systemctl start mysqld
```

登陆服务端**(slave2)**

```shell
grep password /var/log/mysqld.log
mysql -uroot -p

```

设置mysql密码安全策略**(slave2)**

```shell
set global validate_password_policy=0;
set global validate_password_length=4;
alter user 'root'@'localhost' identified by '123456';
\q
```

设置远程登陆**(slave2)**

```shell
mysql -uroot -p123456
create user 'root'@'%' identified by '123456';
grant all privileges on *.* to 'root'@'%' with grant option;
flush privileges;
```

## 安装Hive

解压hive**(master)**

```shell
mkdir -p /usr/hive
tar -zxvf /opt/soft/apache-hive-2.1.1-bin.tar.gz -C /usr/hive
scp -r /usr/hive root@slave1:/usr/
```

复制mysql.jar到slave1中**(connector所在主机)**

```shell
scp /opt/soft/mysql-connector-java-5.1.5-bin.jar root@slave1:/usr/hive/apache-hive-2.1.1-bin/lib
```

编辑hive-env.sh文件**(master,slave1)**

```shell
cd /usr/hive/apache-hive-2.1.1-bin/conf/
cp hive-env.sh.template hive-env.sh
vim hive-env.sh

HADOOP_HOME=/usr/hadoop/hadoop-2.7.3
```

编辑hive-site.xml**(slave1)**

```shell
vim /usr/hive/apache-hive-2.1.1-bin/conf/hive-site.xml

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

从hive的lib包中拷贝jline到hadoop的lib中**(master)**

```shell
cp /usr/hive/apache-hive-2.1.1-bin/lib/jline-2.12.jar /usr/hadoop/hadoop-2.7.3/share/hadoop/yarn/lib
```

编辑hive-site.xml**(master)**

```shell
vim /usr/hive/apache-hive-2.1.1-bin/conf/hive-site.xml

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

配置环境变量**(master,slave1)**

```shell
vim /etc/profile

#set hive environment
export HIVE_HOME=/usr/hive/apache-hive-2.1.1-bin
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile
```

启动hive**(slave1)**

```shell
$HIVE_HOME/bin/hive --service metastore
```

启动hive**(master)**

```shell
$HIVE_HOME/bin/hive
```

