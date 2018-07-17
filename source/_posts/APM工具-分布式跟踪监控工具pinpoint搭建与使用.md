---
title: APM工具-分布式跟踪监控工具pinpoint搭建与使用
date: 2018-07-09 10:34:13
tags:
---

在公司实习期间，主管让我了解一下pinpoint的搭建和使用细则。然后我就根据这个关键词开始了我的面向谷歌编程。

# 配置与搭建

https://www.cnblogs.com/yyhh/p/6106472.html#yy01
https://blog.csdn.net/qq_21816375/article/details/80455681

看了很多博客，各自有各自的坑，我在这里做了一个精华总结版本。

## 环境要求

 1. jdk1.8 --- Java运行环境
 2. hbase-1.0 --- 数据库，用来存储监控信息
 3. tomcat8.0 --- Web服务器
 4. pinpoint-collector.war --- pp的控制器
 5. pinpoint-web.war --- pp展示页面

[pp-collector和pp-web资源下载][1]

## hbase的安装
1.首先安装hbase，根据系统版本选择合适的hbase（我使用的是hbase-1.2.4）

> wget http://archive.apache.org/dist/hbase/1.2.6/hbase-1.2.6-bin.tar.gz
速度较慢，建议下载后上传到服务器

1.解压

> tar -zxvf hbase-1.2.6-bin.tar.gz

2.启动并初始化hbase
编辑hbase-1.2.4/conf/hbase-env.sh,添加JAVA_HOME配置：

> export JAVA_HOME=/root/jdk1.8.0_144

3.编辑hbase-site.xml:

```xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>file:///Users/tangliu/Tmp/hbase</value>
    </property>
    <property>
         <name>hbase.zookeeper.property.dataDir</name>
        <value>/Users/tangliu/Tmp/zookeeper</value>
    </property>
</configuration>
```
分别表示数据和zookeeper数据存放地点。这样配置是本地单实例模式启动，具体和集群配置可以参考hbase官网。

4.   ./hbase-1.2.4/bin/start-hbase.sh启动
5.    ./hbase-1.2.4/bin/hbase shell hbase-create.hbase这是初始化pinpoint需要的表,在git上的位置上可以下载到
6.    验证页面：http://localhost:16010/master-status

## 坑1：这里正常后，页面应该会有显示table

![此处输入图片的描述][2]
确保有显示表，没有的话不用往下做了。先解决这个bug，一般是hbase表的问题，可能是hbase和zookeeper的不一致，需要将hbase和zk里的表都删除，然后再重新导入一次表。

## 安装pinpoint-collector
其实就是，将collector的war包放到一个tomcat容器中的ROOT位置。将tomcat重命名为pp-col，然后再配置一下pp-col的端口。（我选择了tomcat端口全部加1，如18080）

```
cd /data/service/pp-col/conf/
sed -i 's/port="8005"/port="18005"/g' server.xml
sed -i 's/port="8080"/port="18080"/g' server.xml
sed -i 's/port="8443"/port="18443"/g' server.xml
sed -i 's/port="8009"/port="18009"/g' server.xml
sed -i 's/redirectPort="8443"/redirectPort="18443"/g' server.xml
sed -i "s/localhost/`ifconfig eth0 | grep 'inet addr' | awk '{print $2}' | awk -F: '{print $2}'`/g" server.xml
```
部署pinpoint-collector.war包


```
cd /home/pp_res/
rm -rf /data/service/pp-col/webapps/*
tar -zxvf pinpoint-collector-1.5.2.war -d /data/service/pp-col/webapps/ROOT
```
启动tomcat，看一下日志是否启动成功
`tail -f ../logs/catalina.out`

## 安装pinpoint-web（用于展示）
原理和col全部相同，不同的地方只有放在ROOT位置的包不同，以及配置的端口不同（我选择了tomcat端口全部加2，如28080）


# conf中的一些配置参数

 - hbase.properties 配置我们pp-web从哪个数据源获取采集数据，这里我们只指定Hbase的zookeeper地址。
 - jdbc.properties pp-web连接自身Mysql数据库的连接认证配置。 
 - sql目录 pp-web本身有些数据需要存放在MySQL数据库中，这里需要初始化一下表结构。
 - pinpoint-web.properties 这里pp-web集群的配置文件，如果你需要pp-web集群的话。
 - applicationContext-* .xml 这些文件在后续的调优工作中会用到。
 - log4j.xml 日志相关配置。

 同样的，配置好后启动并查看是否启动。
 这时候我们可以访问一下这个地址，在浏览器中输入"http://localhost:28080"，就会出现主页面了。如果访问不了的话，看一下端口是否开放，或者关闭防火墙。

 
## 部署pp-agent采集监控数据
相关说明请参考[Pinpoint Agent][3]

将包解压在service目录下，命名为pp-agent，此处是不需要容器的。
```
cd /data/service
cp /root/demo/pinpoint/agent/target/pinpoint-agent-1.7.3.tar.gz .
mkidr pp-agent
cd pp-agent
tar -zvxf pinpoint-agent-1.7.3.tar.gz

vi pinpoint.config
profiler.collector.ip= your ip
```

## 部署pp-test作为测试用例

这个需要用户在服务启动时添加上这个三行就可以了，安插探针
修改tomcat的catalina.sh文件
```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data/service/pp-agent/pinpoint-bootstrap-1.7.3.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=qinzhao-ID"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=qinzhao"
```



监控Tomcat ,启动
```
./startup.sh
tail -f ../logs/catalina.out
```
然后能看到tomcat启动页面。

最后访问 ip:28080 ,可以看到pinpoint的web界面了。

选择监控应用，并开始监控。
![此处输入图片的描述][4]

## 坑2 没法选择应用
如果没法选择应用的话，多半是数据库hbase出问题了，去确认一下表是否齐全。此外pp-agent的部署是否正确。


  [1]: https://github.com/naver/pinpoint/releases/tag/1.7.3
  [2]: http://paxblmm0h.bkt.clouddn.com/pp-1.png
  [3]: http://naver.github.io/pinpoint/installation.html#installation-2
  [4]: http://paxblmm0h.bkt.clouddn.com/pp-2.png