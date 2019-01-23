# Zookeeper 配置文件

修改主机名到IP地址的映射

```shell
vim /etc/hosts
192.168.61.134 master master.root
192.168.61.135 slave1 slave1.root
192.168.61.136 slave2 slave2.root
```

conf/zoo.cfg配置

```shell
dataDir=/usr/zookeeper/zookeeper-3.4.10/zkdata
dataLogDir=/usr/zookeeper/zookeeper-3.4.10/zkdatalog

server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888

#zookeeper-3.4.10目录下创建zkdata和zkdatalog文件夹
```

zkdata下配置(maste:1,slave1:2,slave2:3)

```shell
vim myid

1
```

配置环境变量

```shell
vim /etc/profile

#set zookeeper environment
export ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.10
PATH=$PATH:$ZOOKEEPER_HOME/bin

source /etc/profile
```

各个主机上启动zookeeper

```shell
$ZOOKEEPER_HOME/bin/zkServer.sh start
```

查看状态

```shell
$ZOOKEEPER_HOME/bin/zkServer.sh status
```

