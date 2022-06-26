|                   | Canal  | Maxwell                    |
| ----------------- | ------ | -------------------------- |
| 语言              | Java   | Java                       |
| 活跃度            | 活跃   | 活跃                       |
| HA                | 支持   | 定制，但是支持断点还原功能 |
| 数据落地          | 定制   | Kafka                      |
| 分区              | 支持   | 支持                       |
| bootstrap（引导） | 不支持 | 支持                       |
| 数据格式          | 自由   | json                       |
| 文档              | 详细   | 详细                       |
| 随机读            | 支持   | 支持                       |

- J 选择Maxwell原因
  - 服务端和客户端一起，轻量级
  - 支持断点还原功能+bootstrap+json

# binlog

- 查找binlog配置文件地址

```shell
# 1 找到my.cnf
select @@basedir;

# 2 如果还是找不到文件的位置，可以执行下面命令，可以得到mysql的配置文件的默认加载顺序
mysql --help | grep 'Default options' -A 1
## Default options are read from the following files in the given order:
## /etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
```
- 修改my.cnf
```shell
# binlog
#设置唯一id
server-id=1
##开启bin-log，产生的bin-log文件名即为bin-log.*
log-bin=mysql-bin
##指定bin-log为row类别，其他两种是statement、mixed
binlog_format=row
##对指定的数据库开启bin-log，这里是对test数据库开启bin-log服务
binlog-do-db=test
#设置binlog清理时间
expire_logs_days = 30
#binlog每个日志文件大小
max_binlog_size = 100m
#binlog缓存大小
binlog_cache_size = 4m
#最大binlog缓存大小
max_binlog_cache_size = 512m
```
- 重启MySQL服务
```shell
service mysqld restart
```
- 查看

```sql
SHOW VARIABLES LIKE 'log_bin';

-- 查看日志开启状态 
show variables like 'log_%';
-- 查看所有binlog日志列表
show master logs;
-- 查看最新一个binlog日志的编号名称，及其最后一个操作事件结束点 
show master status;
-- 刷新log日志，立刻产生一个新编号的binlog日志文件，跟重启一个效果 
flush logs;
-- 清空所有binlog日志 
reset master;
```
- 看日志

```shell
cd /var/lib/mysql
mysqlbinlog --no-defaults mysql-bin.000002
```



# canal

- https://github.com/alibaba/canal/
```shell
# 下载 
wget https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.deployer-1.1.5.tar.gz

mkdir canal1.1.5
cd /home/hadoop/app/canal/example
vim instance.properties
```
- instance.properties
```conf
# 数据库地址
canal.instance.master.address=127.0.0.1:3306
# binlog日志名称
canal.instance.master.journal.name=mysql-bin.000001
# mysql主库链接时起始的binlog偏移量
canal.instance.master.position=154
# mysql主库链接时起始的binlog的时间戳
canal.instance.master.timestamp=
canal.instance.master.gtid=

# username/password
# 在MySQL服务器授权的账号密码
canal.instance.dbUsername=canal
canal.instance.dbPassword=Canal@123456
# 字符集
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false

# table regex .*\\..*表示监听所有表 也可以写具体的表名，用，隔开
canal.instance.filter.regex=.*\\..*
# mysql 数据解析表的黑名单，多个表用，隔开
canal.instance.filter.black.regex=
```
- kafka
```shell
# 创建
kafka-topics.sh --create --zookeeper hadoop000:2181/kafka --replication-factor 1 --partitions 1 --topic canal_test

# 查看
kafka-topics.sh --describe --zookeeper hadoop000:2181/kafka --topic canal_test
```


# maxwell
- https://maxwells-daemon.io/

- https://github.com/zendesk/maxwell



- 安装部署

```shell
# 修改MySQL my.cnf配置

# 下载
wget https://github.com/zendesk/maxwell/releases/download/v1.14.7/maxwell-1.14.7.tar.gz

# 解压 tar -zxvf xx -C .

# 配置
vim ~/.bash_profile
```

- MySQL

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'maxwell'@'%' IDENTIFIED BY 'Maxwell@123' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```



- producer
  - Kafka
    - 修改配置
      - kafka_version：run maxwell with specified kafka producer version. <font color=red>Not available in config.properties.</font>
      - init_position
      - jar
  
  ```shell
  mv config.properties.example config.properties
  vim config.properties
  ```
  
  
  
  - stdout(用于测试)

```shell
maxwell --user='maxwell' --password='Maxwell@123' --host='127.0.0.1' --producer=stdout
```

输出数据

```json
# 测试 1
insert into person values(5,"pk5");
{"database":"test","table":"person","type":"insert","ts":1624958371,"xid":95262,"commit":true,"data":{"id":5,"name":"pk5"}}

update person set name='ssss' where id=2;
{"database":"test","table":"person","type":"update","ts":1624958927,"xid":95606,"commit":true,"data":{"id":2,"name":"ssss"},"old":{"name":"xxx2"}}
```

- kafka

```shell
bin/maxwell --user='maxwell' --password='Maxwell@123' --host=hadoop000 --producer=kafka --kafka.bootstrap.servers=hadoop000:9092 --kafka_topic=maxwell
```



- 遇到的问题
  - 特定场合 大小写
    - https://github.com/zendesk/maxwell/pull/909
  - 时区
  - column与value不相等，串位。。

# MySQL知识点

## 权限

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;  
GRANT ALL PRIVILEGES ON test.* TO 'flinkcdc'@'%' IDENTIFIED BY 'Flinkcdc@123' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

- reload
  - 这类权限的授权不是针对某个数据库的，因此须使用on *.*来进行
  - reload 是 administrative 级的权限，即 server administration；这类权限包括：
```mysql
   CREATE USER, PROCESS, RELOAD, REPLICATION CLIENT, REPLICATION SLAVE, SHOW DATABASES, SHUTDOWN, SUPER
```
