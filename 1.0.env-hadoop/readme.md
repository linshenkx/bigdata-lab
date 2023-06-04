## 零 前言
本部分记录实验环境部署方法。
参考 https://github.com/big-data-europe/docker-hadoop/tree/master 。
使用docker部署hadoop集群。
有namenode、datanode、resourcemanager、nodemanager、historyserver各一个节点。
宿主机环境为: win11、wsl2、dockerDesktop

命令操作默认为wsl环境下执行，docker相关操作可以在windows下执行

## 一 修改配置文件
默认配置文件提供给yarn的计算资源为8c16g，可根据实际机器资源和需求修改
yarn-nodemanager的可用计算资源以及mapreduce任务的资源配置项如下
```shell
YARN_CONF_yarn_nodemanager_resource_memory___mb=16384
YARN_CONF_yarn_nodemanager_resource_cpu___vcores=8

MAPRED_CONF_mapred_child_java_opts=-Xmx4096m
MAPRED_CONF_mapreduce_map_memory_mb=4096
MAPRED_CONF_mapreduce_reduce_memory_mb=8192
MAPRED_CONF_mapreduce_map_java_opts=-Xmx3072m
MAPRED_CONF_mapreduce_reduce_java_opts=-Xmx6144m
```
## 二 启动
```shell
docker-compose up -d
```
## 三 修改本地hosts文件
添加如下记录
```shell
127.0.0.1 namenode
127.0.0.1 datanode
127.0.0.1 resourcemanager
127.0.0.1 nodemanager
127.0.0.1 historyserver

```
## 四 在浏览器中访问对应界面
1. namenode: http://namenode:9870
2. datanode: http://datanode:9864
3. resourcemanager: http://resourcemanager:8088
4. nodemanager: http://nodemanager:8042
## 五 获取连接配置文件
得到用于连接的xml配置文件
```shell
rm -rf hadoop321-conf
docker cp namenode:/opt/hadoop-3.2.1/etc/hadoop hadoop321-conf
find hadoop321-conf/* ! -name "*.xml" | xargs rm -rf
ls -lh hadoop321-conf

```
## 六 配置本地访问客户端
hadoop服务器版本为3.2.1，这里是客户端版本，不一定要一致

### 1-a curl下载
即使配置代理，curl下载仍很慢，建议手动下载，再复制解压到/opt目录下
```shell
curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS
gpg --import KEYS
export HADOOP_VERSION=3.3.4
export HADOOP_URL=https://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz
curl -fSL "$HADOOP_URL" -o /tmp/hadoop.tar.gz \
    && curl -fSL "$HADOOP_URL.asc" -o /tmp/hadoop.tar.gz.asc \
    && gpg --verify /tmp/hadoop.tar.gz.asc \
    && tar -xvf /tmp/hadoop.tar.gz -C /opt/ \
    && rm /tmp/hadoop.tar.gz*

```
### 1-b 手动下载
手动下载到/tmp目录下
```shell
mv /tmp/hadoop-3.2.1.tar.gz /opt/
cd /opt/
tar -zxf hadoop-3.2.1.tar.gz
rm -rf hadoop-3.2.1.tar.gz

```
### 2 配置环境变量
```shell
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export PATH=${PATH}:${HADOOP_HOME}/bin

```

### 3 复制连接配置文件
```shell
mv hadoop321-conf/* ${HADOOP_HOME}/etc/hadoop/
rm -rf hadoop321-conf

```


## 七 本地测试
```shell
hadoop fs -mkdir -p /tmp
hadoop fs -put -f $HADOOP_HOME/README.txt /tmp

hadoop fs -rm -r /tmp/output
yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar  wordcount /tmp/README.txt /tmp/output
hadoop fs -ls /tmp/output

yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar  pi 16 10000

```