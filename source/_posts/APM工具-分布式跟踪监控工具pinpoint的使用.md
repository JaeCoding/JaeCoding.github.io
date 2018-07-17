---
title: APM工具-分布式跟踪监控工具pinpoint的使用
date: 2018-07-09 10:34:13
tags: [工具部署,apm监控]
categories: 技术博客

---

#APM工具-分布式跟踪监控工具pinpoint的使用

# 使用前提
前提是pinpoint-collector和pinpoint-web已经部署在服务器上，具体部署可参考。
[参考1][1] [参考2][2]

所监控的客户端（pp-test）只需要配置两个位置。


# 部署pp-agent采集监控数据
[下载地址][3]，只需要下载pp-agent

下载较慢的话可以从我的源这[下载][4]

相关参数说明请参考[Pinpoint Agent][5]

将包解压在service目录下（位置随意，方便查找即可），命名为pp-agent，此处是不需要容器的。作为监控应用的采集器
```
cd /data/service
cp /root/demo/pinpoint/agent/target/pinpoint-agent-1.7.3.tar.gz .
mkidr pp-agent
cd pp-agent
tar -zvxf pinpoint-agent-1.7.3.tar.gz
```
改采集ip为采集应用的服务器的ip（安装了pp-col的服务器ip）
```
vi pinpoint.config
profiler.collector.ip= 安装了pp-col服务器的ip
```
主要修改IP，只需要指定到安装pp-col的IP就行了，安装pp-col启动后，自动就开启了9994，9995，9996的端口了。这里就不需要操心了，如果有端口需求，要去pp-col的配置文件("pp-col/webapps/ROOT/WEB-INF/classes/pinpoint-collector.properties")中，修改这些端口

# 部署pp-test作为测试用例（所监控的应用）

这个需要用户在服务启动时添加上这个三行就可以了，安插探针

修改tomcat的catalina.sh文件
在20行左右增加如下字段

```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data/service/pp-agent/pinpoint-bootstrap-1.7.3.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=ID"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=appName"
```

第一行是pp-agent的jar包位置（关键）
第二行是agent的ID，这个ID是唯一的，我是用pp + 今天的日期命名的，只要与其他的项目的ID不重复就好了
第三行是采集项目的名字，这个名字可以随便取，只要各个项目不重复就好了


启动所需要监控的应用
```
./startup.sh
```
查看日志，确实启动了
```
tail -f ../logs/catalina.out
```
访问8080端口，然后能看到tomcat启动页面。


## 使用参考
访问部署pp- ip:28080 ,可以看到pinpoint的web界面了。

左上角选择所监控的应用。右上选择监控时间段与更新间隔。

选择监控应用，并开始监控。
![此处输入图片的描述][6]

鼠标可以在右上角拉框选取。会弹出各访问信息。
![此处输入图片的描述][7]

点击右上Inspector可进入应用详细监控界面。
能够查看内存，cpu，线程，反应时间等


[参考使用教程博客][8]

警报功能：
[APM监控--Pinpoint扩展报警][9]


  [1]: https://www.cnblogs.com/yyhh/p/6106472.html#yy05
  [2]: https://blog.csdn.net/qq_21816375/article/details/80455681
  [3]: https://github.com/naver/pinpoint/releases
  [4]: http://paxblmm0h.bkt.clouddn.com/pinpoint-1.7.7z
  [5]: http://naver.github.io/pinpoint/installation.html#installation-2
  [6]: http://paxblmm0h.bkt.clouddn.com/pp-2.png
  [7]: http://paxblmm0h.bkt.clouddn.com/pp-3.png
  [8]: https://blog.csdn.net/kangguang/article/details/77290209
  [9]: https://blog.csdn.net/xvshu/article/details/79814549#t5