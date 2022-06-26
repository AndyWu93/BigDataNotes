# 基础知识

- https://zookeeper.apache.org/

- 协调、管理
- 分布式系统

- Cluster: 统一的对外处理的服务
  - 可以是以下服务
    - ​	hadoop000:8020
      ​	hdfs fs -put wc.data /input
      ​	hdfs fs -get /input/wc/data
      ​	sc.textFile("/input")

- Leader
- Follower



- 选举  心跳  超时下线

- [下载](https://archive.apache.org/dist/)

# 部署

- 1 JDK8
- 2 配置HOME目录
- 3 conf下 修改配置文件

```shell
cd conf
cp zoo_sample.cfg zoo.cfg

## zoo.cfg 修改一个
dataDir=/home/hadoop/tmp/zookeeper
# 开启四字命令
4lw.commands.whitelist=*
```

- 重要参数

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
```



## 启动

- QuorumPeerMain进程 服务端

```shell
zkServer.sh start/stop/restart/status

# 启动
zkServer.sh start

# status
zkServer.sh status

# 启动客户端
zkCli.sh 
```

- 客户端

```shell
./zkCli.sh

# 查看 / 节点
ls /

# 四字命令
stat /zookeeper/quota

# 节点状态信息
cZxid = 0x0                          # Zxid： zk transcation id 
ctime = Thu Jan 01 08:00:00 CST 1970 # 创建时间
mZxid = 0x0                          # 修改后id
mtime = Thu Jan 01 08:00:00 CST 1970 # 修改时间
pZxid = 0x0                          
cversion = 0
dataVersion = 0                      # znode版本 数据版本号 0是第一个
aclVersion = 0
ephemeralOwner = 0x0                 # 判断临时节点 一长串就是临时节点
dataLength = 0
numChildren = 0                      # 有几个子节点
```

- 创建节点

```shell
create /rz rz-data
get /rz  # 获取节点数据

# 临时节点
create -e /rz/tmp rzdata

# 有序节点
## 多层不能同时创建
create -s /rz/seq seq
```

- 修改

```shell
set /rz new-data
```

- 删除

```shell
delete /rz/tmp
```

- 设置监听

```shell
# 一旦znode里的东西发生变化，就会自动感知
stat -w /rz01

get -w /rz01

ls -w /rz
# 监听父节点 子节点变化 监听不到
```

- 运维

```shell
# nc 命令安装
sudo yum -y install nmap-ncat
# 先在配置中开启四字命令

echo stat | nc hadoop000 2181

# are you OK?
echo ruok | nc hadoop000 2181

# 配置
echo conf | nc hadoop000 2181

# 监听个数
echo wchs | nc hadoop000 2181

# 
echo dump | nc hadoop000 2181
```

# ZK存储的数据模型

- 层次化的目录结构

- 每个节点在ZK中叫做znode   Path

  - **临时节点**：session
    - session关闭，会被删除
    - 不能有子节点
    - 有序
  - **永久节点**：
    - 除非手工删除
    - 可以有子节点
    - 有序


## znode特点(每个节点)

- 身份id 
- 版本号
- 修改、删除，如果版本号不匹配，会报错 <font color=red>!!!</font>
- **znode下面可以有子znode**	也可以有数据
- ZK里面的**数据一定是比较小**的 <font color=red>!!!</font>
- 设置权限ACL
- 为znode添加监听器 <font color=red>!!!</font>
  - znodea进行监听，一旦znodea里面的“东西”发生了变化，就能够自动感知
  - 特性：监听器只能使用一次（新版本：可以监听多个）
    - NodeCreated
    - NodeDataChanged
    - NodeDeleted
    - NodeChildrenChanged
      	

#### The Four Letter Words[#](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_4lw)



# 代码

### 1 原生API

```java
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.List;

public class ZKUtils {
    public static ZooKeeper getZK(String connect, int timeout) throws IOException {
        return new ZooKeeper(connect, timeout,null);
    }

     public static String createNode(String path, String value, ZooKeeper zk) throws Exception {
        return zk.create(path, value.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    public static String getData(String path, ZooKeeper zk) throws Exception {
        return new String(zk.getData(path,false,null));
    }

    public static void update(String path, String value, ZooKeeper zk) throws Exception {
        zk.setData(path, value.getBytes(), -1);
    }

    public static boolean exists(String path, ZooKeeper  zk) throws Exception {
        final Stat stat = zk.exists(path, false);
        return stat != null;
    }


    public static List<String> getChildren(String path, ZooKeeper  zk) throws Exception {
        return zk.getChildren(path, false);
    }

    public static void delete(String path, ZooKeeper zk) throws Exception {
        zk.delete(path, -1);
    }
}
```

#### 测试

```java
import org.apache.zookeeper.ZooKeeper;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import java.io.IOException;
import static org.junit.Assert.*;

public class ZKUtilsTest {

    ZooKeeper zk = null;
    String connect = "hadoop000:2181";
    int timeout = 5000;
    String path = "/rz";

    @Before
    public void setUp() throws IOException {
        zk = new ZooKeeper(connect, timeout, null);
    }

    @Test
    public void testInit() throws Exception {
        // assertNotNull(zk);
        // ZKUtils.createNode(path, "data", zk);
        String data = ZKUtils.getData(path, zk);
        System.out.println(data);
    }
  
    @Test
    public void testInit2() throws Exception {
        for(int i=0 ; i<10 ; i++) {
            String p = path + "/test" + i;
            String value = "数据" + i;
            ZKUtils.createNode(p, value, zk);
        }
    }

    @After
    public void tearDown() throws Exception {
        if (zk != null) {
            zk.close();
        }
    }
}
```

#### IDEA插件

Zoolytic

#### pom.xml 

```xml
 <!-- zookeeper 3.6.2 -->
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>${zk.version}</version>
</dependency>
```



### 2 Curator 操作ZK[#](https://curator.apache.org/)

#### 代码

```java
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.CuratorWatcher;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.retry.RetryNTimes;
import org.apache.curator.retry.RetryOneTime;
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.List;

public class TestCuratorZKUtils {

    CuratorFramework client = null;
    String connect = "hadoop000:2181";
    int timeout = 5000;
    String path = "/rz";
    String namespace = "ruozedata-ws";

    @Before
    public void setUp() throws Exception {

        client = CuratorFrameworkFactory.builder()
                .connectString(connect)
                .sessionTimeoutMs(timeout)
//                .retryPolicy(new RetryOneTime(1000))
//                .retryPolicy(new RetryNTimes(3, 1000))
                .retryPolicy(new ExponentialBackoffRetry(1000, 5))
                .namespace(namespace)
                .build();
        client.start();
    }

    @Test
    public void testInit() throws Exception{
//        client.create()
//                .creatingParentsIfNeeded()
//                .withMode(CreateMode.PERSISTENT)
//                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
//                .forPath(path, "rz".getBytes());
//
//        client.setData().forPath(path, "new-data".getBytes());

//        final Stat stat = new Stat();
//        final byte[] content = client.getData().storingStatIn(stat).forPath(path);
//        System.out.println(path + "内容是:" + new String(content) + ",版本是:"+ stat.getVersion());

//        final Stat stat = client.checkExists().forPath(path);

//        final List<String> children = client.getChildren().forPath("/");
//        for(String child : children) {
//            System.out.println(child);
//        }

//        client.delete().deletingChildrenIfNeeded().forPath("/ruoze");

//        client.getData().usingWatcher(new CuratorWatcher() {
//            @Override
//            public void process(WatchedEvent event) {
//                System.out.println("监听到:" + event.getPath());
//            }
//        }).forPath(path);

        final NodeCache nodeCache = new NodeCache(client, path);
        nodeCache.start();

        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                String data = new String(nodeCache.getCurrentData().getData());
                System.out.println(data);
            }
        });

        Thread.sleep(100000000);
    }

    @After
    public  void tearDown() throws Exception {
    }
}
```



#### pom.xml

```xml
<dependency>
   <groupId>org.apache.curator</groupId>
   <artifactId>curator-framework</artifactId>
   <version>5.1.0</version>
</dependency>
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-recipes</artifactId>
  <version>5.1.0</version>
</dependency>
```

​	
​	

# 分布式锁基于ZK

Client1:

- 创建一个临时顺序节点
- 判断是否加锁：依据是 是不是最小的
  - ok => 加锁
- 业务处理完：释放锁

Client：

- 创建一个临时顺序节点
- 判断是否加锁：依据是 是不是最小的
  - 不是最小的，找到他的上一个节点 加一个监听器

![](pictures/zk/分布式锁.jpeg)

​	
​	
​	
​	
​	
​	






​	
​	
​	
​	




​	
​	
​	
​	
​	
​	