# 1、阿里云 

- 按量付费机器 3台 关机选择不收费模式 否则扣钱很狠

# 2、CDH全局

- Cluster ：HDFS YARN ZOOKEEPER.....
- CMS: 5个服务 
- 元数据: MySQL 

## 平台

- Cloudera 出的大数据平台 CDH
- CM(server+agent) + Cluster+ CMS

# 3、卸载

- 对应加密视频 

# 4、部署 （模式: tar+http）

| cm部署  | parcel部署 | 版本      |
| ------- | ---------- | --------- |
| rpm安装 | http服务   | CDH6.3.1  |
| tar解压 | 非http服务 | CDH5.16.1 |
|         |            |           |

## 4种部署模式，优先

- rpm安装       http服务
- tar解压       http服务

## 准备工作

- 4.1 阿里云 按量付费机器 3台
- 4.2 三台机器hosts文件要配置  内网ip 机器名称
- 4.3 jdk部署 
- 4.4 mysql部署
  4.5 模式的确定  

create database cmf DEFAULT CHARACTER SET utf8;
create database amon DEFAULT CHARACTER SET utf8;
grant all on cmf.* TO 'cmf'@'%' IDENTIFIED BY 'Ruozedata123456!';
grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'Ruozedata123456!';
flush privileges;


1.1.所有节点创建录及解压
mkdir /opt/cloudera-manager
tar -zxvf /root/cdh/cloudera-manager-centos7-cm5.16.2_x86_64.tar.gz -C /opt/cloudera-manager/

1.2.所有节点修改agent的配置，指向server的节点ruozedata001
sed -i "s/server_host=localhost/server_host=ruozedata001/g" /opt/cloudera-manager/cm-5.16.2/etc/cloudera-scm-agent/config.ini

1.3.主节点修改server的配置:
vi /opt/cloudera-manager/cm-5.16.2/etc/cloudera-scm-server/db.properties 
com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=ruozedata001
com.cloudera.cmf.db.name=cmf
com.cloudera.cmf.db.user=cmf
com.cloudera.cmf.db.password=Ruozedata123456!
com.cloudera.cmf.db.setupType=EXTERNAL

1.4.所有节点创建⽤户
useradd --system --home=/opt/cloudera-manager/cm-5.16.2/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

1.5.⽬录修改⽤户及⽤户组
chown -R cloudera-scm:cloudera-scm /opt/cloudera-manager


[root@ruozedata001 cdh]# ll /var/www/html/cdh5_parcel
total 2082872
-rw-r--r-- 1 root root 2132782197 May 15 14:11 CDH-5.16.2-1.cdh5.16.2.p0.8-el7.parcel
-rw-r--r-- 1 root root         41 May 15 14:14 CDH-5.16.2-1.cdh5.16.2.p0.8-el7.parcel.sha
-rw-r--r-- 1 root root      66804 May 15 14:13 manifest.json
[root@ruozedata001 cdh]# 


坑: 其他机器通过curl命令去访问http路径 是否通畅


启动检查日志
[root@ruozedata001 cloudera-scm-server]# pwd
/opt/cloudera-manager/cm-5.16.2/log/cloudera-scm-server
[root@ruozedata001 cloudera-scm-server]# ll
total 388
-rw-r----- 1 root root 201135 May 15 21:24 cloudera-scm-server.log
-rw-r--r-- 1 root root 185896 May 15 21:24 cloudera-scm-server.out
-rw-r----- 1 root root    623 May 15 21:24 cmf-server-perf.log
[root@ruozedata001 cloudera-scm-server]# 


错误:
com.cloudera.parcel.ParcelException: Invalid local parcel repository path configured: /opt/cloudera/parcel-repo
        at com.cloudera.parcel.components.LocalParcelManagerImpl.getParcelRepo(LocalParcelManagerImpl.java:394)
        at com.cloudera.parcel.components.ParcelDownloaderImpl$1.run(ParcelDownloaderImpl.java:463)
        at com.cloudera.parcel.components.ParcelDownloaderImpl$1.run(ParcelDownloaderImpl.java:459)
        at com.cloudera.cmf.persist.ReadWriteDatabaseTaskCallable.call(ReadWriteDatabaseTaskCallable.java:36)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
2021-05-15 21:29:54,413 INFO ProcessStalenessDetector-0:com.cloudera.cmf.service.config.components.ProcessStalenessDetector: Running staleness check with FULL_CHECK for 0/0 roles.
2021-05-15 21:29:54,413 INFO ProcessStalenessDetector-0:com.cloudera.cmf.service.config.components.ProcessStalenessDetector: Staleness check done. Duration: PT0.002S 
2021-05-15 21:29:54,413 INFO ProcessStalenessDetector-0:com.cloudera.cmf.service.config.components.ProcessStalenessDetector: Staleness check execution stats: average=0ms, min=0ms, max=0ms.
2021-05-15 21:30:33,790 INFO avro-servlet-hb-processor-0:com.cloudera.server.common.AgentAvroServlet: (11 skipped) AgentAvroServlet: heartbeat processing stats: average=11ms, min=3ms, max=181ms.
^Z
[3]+  Stopped                 tail -200f cloudera-scm-server.log
[root@ruozedata001 cloudera-scm-server]# mkdir -p /opt/cloudera/parcel-repo
[root@ruozedata001 cloudera-scm-server]# chown -R cloudera-scm:cloudera-scm /opt/cloudera/


--CM 部署完成


/opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-server start
/opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-agent start

/opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-server stop
/opt/cloudera-manager/cm-5.16.2/etc/init.d/cloudera-scm-agent stop