### 0 提供数据源
在linux环境执行
```shell
nc -lk 11999

```

### 1 开发
直接在idea中启动main方法即可

### 2 打包
```shell
gradle shadowJar
```
### 3 提交运行

```shell
export HADOOP_CLASSPATH=`hadoop classpath` \
export FLINK_HOME="/opt/flink-1.16.0" \
&& ${FLINK_HOME}/bin/flink run-application \
-t yarn-application \
build/libs/*.jar

```