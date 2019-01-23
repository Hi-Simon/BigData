# Hadoop相关配置文件-命令

## Hadoop Shell 命令

hadoop 格式化

```shell
hadoop namenode -format
```

启动hadoop

```shell
$HADOOP_HOME/sbin/start-all.sh
```

进入安全模式

```shell
hdfs dfsadmin -safemode enter
```

退出安全模式

```shell
hdfs dfsadmin -safemode leave
```

在hadoop上创建文件夹

```shell
hadoop fs -mkdir /"filename"
```

创建文件

```shell
hadoop fs -touchz /'filename'
```

查看根目录下所有文件

```shell
hadoop fs -ls /
```

递归查询子目录所有文件

```shell
hadoop fs -ls -R /
```

复制文件“filename1” 并重命名为”filename2“

```shell
hadoop fs -cp /"filename" /"filename2"
```

移动文件”filename1” 并重命名为”filename2”

```shell
hadoop fs -mv /"filename1" /"filename2"
```

移除文件

```shell
hadoop fs -rm /"filename"
```

清空回收站

```shell
hadoop fs -expunge
```

查看文件大小

```shell
hadoop fs -du -s (-R) /test1/data.txt
```

查看文件内容

```shell
hadoop fs -cat /'filename'
hadoop fs -text /'filename'      //输出文本格式
hadoop fs -tail /'filename' 	 //将文件尾部1K字节的内容输出
```

将本地文件上传到hadoop里

```shell
hadoop fs -put "本地文件路径" "hadoop目标路径"
```

将hadoop文件下载到本地

```shell
hadoop fs -get "hadoop文件路径" ”本地目标路径"
```

改变文件的拥有者为”name”

```shell
hadoop fs -chown name (-R) /test1/data.txt
```

关闭hadoop

```shell
$HADOOP_HOME/sbin/stop-all.sh
```

## Hadoop 配置

系统/etc/profile中加入hadoop环境

```shell
export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3
export CLASS_PATH=$HADOOP_HOME:$HADOOP_HOME/lib
export PATH=$PATH:$HADOOP_HOME/bin

生效：source /etc/profile
```

hadoop-env.sh中加入java环境

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_171
```

core-site.xml配置

```xml
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

yarn-site.xml配置

```xml
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

slaves文件

```xml
slave1
slave2
```

master

```
master
```

hdfs-site.xml配置

```xml
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

mapred-site.xml配置

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```

