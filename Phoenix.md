# 1 概念

OLAP 联机分析处理

OLTP 联机事物处理 关系型数据库 Oracle MySQL

OLTP and OLAP：tidb



[Apache Phoenix](https://phoenix.apache.org/) enables OLTP and operational analytics in Hadoop for low latency applications 

- the power of standard SQL (除了upsert语法) and JDBC APIs with full ACID transaction capabilities
- the flexibility of late-bound, schema-on-read capabilities from the NoSQL world by leveraging HBase as its backing store

- fully integrated with other Hadoop products such as Spark, Hive, Pig, Flume, and Map Reduce



# 2 部署

## 2.1版本选择

- 没能力：直接官网安装包
  - 4.14.0-cdh5.14.2
- 有能力编译
  - phoenix-4.14.1-cdh5.16.1 + kerberos bug修复 和 dbeaver bug修复

## 2.2 部署

- hbase部署
- 重要jar包
  - phoenix-4.14.0-cdh5.14.2-client.jar  客户端 项目代码加载这个 150mb
  - phoenix-4.14.0-cdh5.14.2-server.jar  服务端 与Hbase绑定
  - phoenix-spark-4.14.0-cdh5.14.2.jar spark phoenix 外部数据源

```shell
# 服务端 与Hbase绑定
cp ~/app/apache-phoenix-4.14.0-cdh5.14.2-bin/phoenix-4.14.0-cdh5.14.2-server.jar ~/app/hbase-1.2.0-cdh5.16.2/lib/

# 修改配置
cd /home/hadoop/app/hbase-1.2.0-cdh5.16.2/conf
vim hbase-site.xml

cd /home/hadoop/app/hbase-1.2.0-cdh5.16.2
./bin/stop-hbase.sh
./bin/start-hbase.sh

cp hbase-site.xml hbase-site.xml.bak
ln -s ~/app/hbase-1.2.0-cdh5.16.2/conf/hbase-site.xml hbase-site.xml

# 注意 假如 hdfs ha怎么办？
# cdh：/etc/hdfs/conf/core-site.xml
#      /etc/hdfs/conf/hdfs-site.xml

# 注意 Python版本 2.7
python -V

# 启动
cd /home/hadoop/app/apache-phoenix-4.14.0-cdh5.14.2-bin/bin

./sqlline.py hadoop000:2181
```

- hbase-site.xml

```xml
<property>
  <name>hbase.table.sanity.checks</name>
  <value>false</value>
</property>
<property>
  <name>hbase.regionserver.wal.codec</name>
  <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
</property>
<property>
  <name>phoenix.schema.isNamespaceMappingEnabled</name>
  <value>true</value>
</property>
<property>
  <name>phoenix.schema.mapSystemTablesToNamespace</name>
  <value>true</value>
</property>
```

## 2.3 webUI

- **Requests** 
  - read+1 
  - write+1

# 3 [数据类型](https://phoenix.apache.org/language/datatypes.html)

- INTEGER
- BIGINT
- FLOAT
- DOUBLE
- DECIMAL
- VARCHAR
  - 生产 根据MySQL varchar 数据长度的进行翻倍存储 
  - 因为： 1上游字段可能增长；2 Phoenix不支持modify语法 只能drop再add字段

- CHAR
- TIMESTAMP（针对MYSQL datetime timestamp）

> 加字段好加 add语法
>
> 不支持modify语法 只能drop 再add



# 4 [语法](https://phoenix.apache.org/language/index.html)

- 不区分大小写，统一转化为大写

```sql
-- phoenix 建表必须指定 pk--> hbase rowkey

-- Phoenix的schema => hbase的namespace => database
-- list_namespace(hbase shell)
create schema rzdata;

create table rzdata.test(id bigint not null primary key, name varchar(255), age integer);
!tables

-- 将insert update融合，数据不存在就insert， 数据存在就update
-- scan 'RZDATA:TEST'(hbase shell)
upsert into rzdata.test values(1, 'rz', 18);
upsert into rzdata.test values(2, '狗狗', 16);

-- 覆盖
upsert into rzdata.test values(1, 'rz2', 18);
```

- delete
- drop



## [Schema](https://phoenix.apache.org/namspace_mapping.html)

- From v4.8.0 onwards, user can enable to map it’s schema to the namespace so that any table created with schema will be created in the corresponding namespace of HBase.
- Earlier, every table (with schema or without schema) was created in default namespace.
- **服务端(HBase)和客户端(Phoenix**)同时配置（软链接）hbase-site.xml
- DBeaver也要加属性

```xml
<property>
  <name>phoenix.schema.isNamespaceMappingEnabled</name>
  <value>true</value>
</property>
<property>
  <name>phoenix.schema.mapSystemTablesToNamespace</name>
  <value>true</value>
</property>
<!-- 基于这2个参数 看文档 注意客户端和服务端 哪边配置 -->
```

- shell

```shell
# 创建索引表
create index test_idx on rzdata.test(name,age);

explain select name from rzdata.test where age = 18;    # FULL SCAN OVER RZDATA:TEST_IDX

#  RANGE SCAN OVER RZDATA:TEST_IDX ['rz'] (优) 
## 在索引表 用第一个字段查第二个字段
explain select age from rzdata.test where name = 'rz';
```



# 5 [盐表](https://phoenix.apache.org/salted.html)

- 加盐本质是随机数

- 针对大数据量

- 好处：有助于数据均匀落在各个region-->各个节点，从而提升写的性能

- 坏处：查询性能有所下降

  ```shell
  1-1000 非盐表 数据在一个rs 查询性能高
  1-1000   盐表 数据在多个rs 查询性能低
  
  1-1kw  非盐表 数据在一个rs 查询性能低
  1-1kw    盐表 数据在多个rs 查询性能高
  ```

- 坑：
  - 规划20个rs节点，salt_buckets=20
  - 数据量越来越多，新增50台，sallt_bucket=20-->70
  - 方法：drop表-->create表-->导数据



## Phoenix

- salt_buckets --> 1-256 范围
- 不成文规定，regionserver有多少，就设置多少台(并不一定均匀分布 看master怎么管理)
  - 比如 设为20， 会有20 个region 分散到各个rs节点上（左闭右开）


```sql
create table rzdata.testsalt (
id integer primary key, 
name varchar, 
age integer, 
address varchar
) salt_buckets=20;

upsert into rzdata.testsalt values(1, 'rz', 17, '北疆');
upsert into rzdata.testsalt values(2, 'jj', 17, '上海');

# hbase shell 查看index
scan 'RZDATA:TESTSALT'
```

- hbase shell

```sql
# 比如 设为20， 会有20+1个region 分散到各个rs节点上
# SPLITS 必须大写
create 'testsalt2', 'order', SPLITS => ['a','b','c','d']  # 预分区表

list  # 查看所有表
```



# 6 [二级索引](https://phoenix.apache.org/secondary_indexing.html)

Secondary indexes are an orthogonal way to access data from its primary access path. In HBase, you have a single index that is lexicographically sorted on the primary row key. Access to records in any way other than through the primary row requires scanning over potentially all the rows in the table to test them against your filter. With secondary indexing, the columns or expressions you index form an alternate row key to allow point lookups and range scans along this new axis.



## 6.1 全局索引、本地索引

### 全局索引

- 本质是一张表，对于使用该索引，在查询条件字段如果不在索引里就不走index，除非使用hint（强制）
- <font color=red>重读轻写</font>, 写的QPS没有读的QPS高
- 盐表index 就是 全局索引
- We intercept the data table updates on write ([DELETE](https://phoenix.apache.org/language/index.html#delete), [UPSERT VALUES](https://phoenix.apache.org/language/index.html#upsert_values) and [UPSERT SELECT](https://phoenix.apache.org/language/index.html#upsert_select)), build the index update and then sent any necessary updates to all interested index tables
- 更新路径 clients ==> index table ==>data  table



总结

- where条件 全部是索引字段，select条件 全部是索引字段，最优的
- where条件 全部是索引字段，select条件 部分是索引字段，为了走index，强制
- 多看执行计划，对应调整

```sql
create index testsalt_idx on rzdata.testsalt(name, age);
drop index testsalt_idx on rzdata.testsalt;
-- 查询表
!tables

select * from rzdata.testsalt_idx;
+---------+--------+------+
| 0:NAME  | 0:AGE  | :ID  |
+---------+--------+------+
| j       | 17     | 2    |
| rz      | 17     | 1    |
+---------+--------+------+
# 0 --> Phoenix建表的默认列族 0


# 全表扫描 all
explain select * from rzdata.testsalt;
explain select /*+ INDEX(rzdata.testsalt testsalt_idx) */ * from rzdata.testsalt;  # 强制走索引

# select address
## 全表扫描 all
explain select address from rzdata.testsalt where age=17;
## 强制走index
explain select /*+ INDEX(rzdata.testsalt testsalt_idx) */ address from rzdata.testsalt where age=17;

# 走index表 
## RANGE SCAN OVER RZDATA:TESTSALT_IDX SERVER FILTER BY FIRST KEY ONLY AND TO_INTEGER("AGE") = 17
explain select name from rzdata.testsalt where age=17;
```



### 本地索引

- <font color=red>重写轻读</font>
- **local indexes, index data and table data** co-reside on **same server** preventing any network overhead during writes
- From 4.8.0 onwards we are storing all local index data in the <font color=red>separate shadow column families</font> in the same data table
- 本质：
  - **索引数据和源数据存储在同一个表里，不同的cf**
  - 对于本地索引，查询中无论使用hint强制指定 或者 查询列是否都在index表里，都默认使用索引表
- 总结：
  - 要有where条件，且字段不要以 * 标识

- 好处：
  - 写的性能高，无需写index table（与全局索引比较）

- 坏处：
  - 在使用本地索引时，必须检查所有的region的数据，因为无法先确认索引数据的准确区域位置，所以读的开销较大

```sql
# region 3+1个
# split 是预分区表
create table rzdata.testlocal (
id integer primary key, 
name varchar, 
age integer, 
address varchar
) split on (1,2,3);
 
# 插数据
upsert into rzdata.testlocal values(1, 'rz', 17, '北疆');
upsert into rzdata.testlocal values(2, 'j', 17, '上海');

# 建索引
create local index testlocal_idx on rzdata.testlocal(name,age);


# 全表扫描 all
explain select * from rzdata.testlocal;
explain select /*+ INDEX(rzdata.testlocal testlocal_idx) */ * from rzdata.testlocal;

# 走index RANGE SCAN
explain select address from rzdata.testlocal where age=17;
explain select /*+ INDEX(rzdata.testlocal testlocal_idx) */ address from rzdata.testlocal where age=17;
```

- hbase shell

```sql
describe 'RZDATA:TESTLOCAL'

# 结果 2个cf
COLUMN FAMILIES DESCRIPTION
{NAME => '0', BLOOMFILTER => 'NONE', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'FAST_DIFF', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
{NAME => 'L#0', BLOOMFILTER => 'NONE', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'FAST_DIFF', TTL => 'FOREVER',COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
```



## 6.2 索引比较

- 全局vs本地
  - 假如不清楚业务层场景（模糊）：建议使用 盐表和全局索引
    - 1 盐表语法简单
    - 2 写的性能没有读的性能高，J哥生产上QPS写在5w/s
    - 3 全局是表（重读轻写），本地是cf（重写轻读）
- 预分区表vs盐表
  - id自增长 建议盐表
  - a-z 可以预分区


```shell
# 预分区表 
# 写 不加 ，读 不加
1值     \x80\x80\x00\x01
region  \x80\x80\x00\x01

# 盐表是一种特殊的预分区表
# 写 加前缀，读 去除前缀
\x60\x00\x80\x80\x00
\x60\x80\x00\x00\x01
```



## 6.3 覆盖索引

本质：

- 原始列的数据也存在**索引表**中
- 这样查询索引数据时，不需要去原始表查询，直接获取结果，节省开销

好处

- 空间换时间 查询快

坏处

- 存储空间浪费

```sql
create table rzdata.testinclude
(id integer primary key, 
 name varchar, 
 age integer, 
 address varchar,
 salary double) split on (1,2,3);

upsert into rzdata.testinclude values(1, 'rz', 17, '北疆');
upsert into rzdata.testinclude values(2, 'j', 17, '上海');

create index testinclude_idx on rzdata.testinclude(name,age) include(address);


select * from rzdata.testinclude_idx;
+---------+--------+------+------------+
| 0:NAME  | 0:AGE  | :ID  | 0:ADDRESS  |
+---------+--------+------+------------+
| j       | 17     | 2    | 上海        |
| rz      | 17     | 1    | 北疆        |
+---------+--------+------+------------+


# 全表扫描 all
explain select * from rzdata.testinclude;
explain select address,salary from rzdata.testinclude where age=17;  # salary没在索引表中
explain select /*+ INDEX(rzdata.testinclude testinclude_idx) */ * from rzdata.testinclude;

# 走index
explain select address from rzdata.testinclude where age=17;
explain select address, id from rzdata.testinclude where age=17;
explain select /*+ INDEX(rzdata.testinclude testinclude_idx) */ * from rzdata.testinclude where age=17;
explain select /*+ INDEX(rzdata.testinclude testinclude_idx) */ salary from rzdata.testinclude where age=17;
```



## 6.4 函数索引

```sql
create index upper_name_idx on emp (upper(first_name||' '||last_name))

select emp_id from emp where upper(first_name||' '||last_name)='JOHN'
```

## 6.5 按可变、不可变分类

### 可变索引（分类2）

- 基于表属性设置可变
- index默认就是 可变

### 不可变索引（分类2）

- 基于表属性设置不可变
- index默认就是不可变
- 建表这个参数不加 immutable_rows 不关心
- 场景：append追加， 比如log日志

```sql
create table rzdata.testimmu (
id integer primary key, 
name varchar, 
age integer, 
address varchar
) IMMUTABLE_ROWS=TRUE split on (1,2,3);

create index testimmu_idx on rzdata.testimmu(name,age) include(address);

upsert into rzdata.testimmu values(1, 'rz', 17, '北疆');
upsert into rzdata.testimmu values(2, 'j', 17, '上海');
upsert into rzdata.testimmu values(2, 'j2', 177, '上海');


select * from rzdata.testimmu;
+-----+-------+------+----------+
| ID  | NAME  | AGE  | ADDRESS  |
+-----+-------+------+----------+
| 2   | j     | 17   | 上海      |
| 2   | j2    | 177  | 上海      |
| 1   | rz    | 17   | 北疆      |
+-----+-------+------+----------+

select * from rzdata.testimmu_idx;
+---------+--------+------+------------+
| 0:NAME  | 0:AGE  | :ID  | 0:ADDRESS  |
+---------+--------+------+------------+
| j       | 17     | 2    | 上海        |
| j2      | 177    | 2    | 上海        |
| rz      | 17     | 1    | 北疆        |
+---------+--------+------+------------+
```



## 6.7 异步索引

- **前面的索引都是同步索引**



异步索引应用场景：
- 一次性导入海量数据（历史数据）
- [海量数据方案2](https://phoenix.apache.org/bulk_dataload.html)

```sql
create index async_index on my_schema.my_table(v) async;
```

```shell
${HBASE_HOME}/bin/hbase org.apache.phoenix.mapreduce.index.IndexTool
--schema my_schema
--data-table my_table
--index-table async_idx
--output-path async_idx_hfiles
```



# 7 编译

```shell
mvn clean package -DskipTests
```



# 8 调优(二级索引)

- https://phoenix.apache.org/tuning_guide.html
- https://phoenix.apache.org/tuning.html

调大部分参数(翻倍、3倍、10倍)

- https://phoenix.apache.org/secondary_indexing.html
  - org.apache.phoenix.regionserver.index.handler.count 256/128

# 案例

```shell
./sqlline.py hadoop000:2181
```

- pom

```xml
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-client</artifactId>
    <version>4.14.1-HBase-1.4</version>
    <scope>system</scope>
    <systemPath>/path/to/xx.jar</systemPath>
</dependency>
```

- Phoenix

```sql
-- schema（phoenix） <--> namespace（hbase） <--> database（MySQL）
create schema ruozedata;
-- 建表
create table ruozedata.test
(id bigint not null primary key, name varchar(255), age integer);
-- 加载数据
upsert into ruozedata.test values(1, 'rz', 18);
upsert into ruozedata.test values(2, 'j', 16);

select * from ruozedata.test;

-- 建索引
create index test_idx on ruozedata.test(name,age);

select * from ruozedata.test_idx;

explain select name from ruozedata.test where age = 18;
-- ROUND ROBIN FULL SCAN OVER RUOZEDATA:TEST_IDX
explain select age from ruozedata.test where name = 'rz';
-- RANGE SCAN OVER RUOZEDATA:TEST_IDX ['rz']
-- 生产上建议使用非full scan
```

- hbase

```shell
hbase shell

list_namespace
list_namespace_tables 'RUOZEDATA'
scan 'RUOZEDATA:TEST'
```



- 工具DBeaver
```shell
# 下载客户端
scp phoenix-4.14.0-cdh5.14.2-client.jar
```



## 案例1 Spark-Phoenix

- pom

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <target.java.version>1.8</target.java.version>
  <scala.binary.version>2.12</scala.binary.version>
  <maven.compiler.source>${target.java.version}</maven.compiler.source>
  <maven.compiler.target>${target.java.version}</maven.compiler.target>
  <encoding>UTF-8</encoding>
  <log4j.version>2.12.1</log4j.version>
  <hadoop.version>2.6.3</hadoop.version>
  <spark.version>3.1.1</spark.version>
  <hbase.version>1.2.0</hbase.version>
  <phoenix.version>4.14.0</phoenix.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>

  <dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.14.0-HBase-1.2</version>
  </dependency>
  <dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-spark</artifactId>
    <version>4.14.0-HBase-1.2</version>
  </dependency>

  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>${log4j.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j.version}</version>
  </dependency>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
  </dependency>
</dependencies>
```

- log4j.properties

```properties
log4j.rootLogger=WARN, stdout

log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern= "%d{yyyy-MM-dd HH:mm:ss} %p [%c:%L] - %m%n
```

- code

```scala
import org.apache.hadoop.conf.Configuration
import org.apache.phoenix.spark._
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder().appName(getClass.getSimpleName).master("local[2]").getOrCreate()
val conf = new Configuration()
conf.set("phoenix.schema.isNamespaceMappingEnabled", "true")

// 具体代码

spark.stop()
```

- rdd相关

```scala
// rdd 读数据
spark.sparkContext.phoenixTableAsRDD(
  "rzdata.TESTLOCAL",
  Seq("ID", "NAME", "AGE", "ADDRESS"),
  zkUrl = Some("hadoop000:2181"),
  conf = conf
).foreach(println)

// rdd 写入数据
spark.sparkContext.parallelize(List((3, "aa", 22, "test"), (4, "aa22", 212, "test")))
  .saveToPhoenix(
    "rzdata.TESTLOCAL",
    Seq("ID", "NAME", "AGE", "ADDRESS"),
    zkUrl = Some("hadoop000:2181"),
    conf = conf
  )
```

- SQL相关

```scala
val df = spark.read.format("org.apache.phoenix.spark")
            .option("table", "rzdata.TESTLOCAL")
            .option("zkUrl", "hadoop000:2181")
            .option("phoenix.schema.isNamespaceMappingEnabled", "true")
            .load()
```

