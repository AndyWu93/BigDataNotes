# 大数据中的调度

- crontab	
- oozie：重量级  cm    xml
- azkaban： ^^^^^   azkaban.github.io/
- airflow:  
- ds

# Azkaban版本
​	2.x   不要的
​	3.x  

# 部署

## 编译

### 1 下载文件

```shell
cd /home/hadoop/software/azkaban-3.90.0/gradle/wrapper
wget gradle-4.6-all.zip  # 慢
```

### 2 开始编译

```shell
cd /home/hadoop/software/azkaban-3.90.0
# 大约10min
 ./gradlew build installDist -x test
```

## 部署

- solo-server：学习、测试足够了

- distributed multiple-executor：生产
- 架构
  - Web-server : 提供UI配置
  - Excutor-server：作业运行

```shell
azkaban-3.90.0/azkaban-solo-server/build/distributions/azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz
```

### 启动

```shell
cd /home/hadoop/app/azkaban-solo-server-0.1.0-SNAPSHOT/bin
# 在bin目录下启动 不然会有问题
./start-solo.sh

# vim conf/azkaban.properties
default.timezone.id=Asia/Shanghai

# vim conf/azkaban-users.xml 
# 服务无需重启
<user password="admin" roles="admin" username="admin"/>
```

# Creating Flows

- https://azkaban.readthedocs.io/en/latest/createFlows.html
- basic.flow

```yml
nodes:
  - name: jobA
    type: command
    config:
      command: hadoop dfs -ls /
```

> 如果没有配置hadoop的环境变量，需要写全路径

- flow20.project

```yml
azkaban-flow-version: 2.0
```

### API

- 返回seesionId

```shell
curl -k -X POST --data "action=login&username=admin&password=admin" http://hadoop000:8081
```

pom

```xml
<dependencies>
  <dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>fluent-hc</artifactId>
    <version>4.5.6</version>
  </dependency>

  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.8.1</version>
  </dependency>

  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
  </dependency>

   <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.26</version>
  </dependency>

  <dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpmime</artifactId>
    <version>4.5.6</version>
  </dependency>

  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.58</version>
  </dependency>
</dependencies>
```

