title： Pinpoint技术概述原理
date: 2018-07-09 10:34:13
tags: [pinpoint]
category: [经验总结]

---

# Pinpoint技术概述
翻译自官方文档 Technical Overview Of Pinpoint

Pinpoint是大型分布式系统服务的n层架构跟踪平台。 ，提供解决方案来处理海量跟踪数据。2012年七月开始开发，2015年1月9日作为开源项目启动。

Pinpoint的特点如下:

 - 分布式事务跟踪， 跟踪跨分布式应用的消息
 - 自动检测应用拓扑，帮助你搞清楚应用的架构
 - 水平扩展  以便支持大规模服务器集群
 - 提供代码级别的可见性  以便轻松定位失败点和瓶颈
 - 使用字节码增强技术，添加新功能 而无需修改代码

本文将介绍Pinpoint： 什么促使我们开始搭建它， 用了什么技术， 还有Pinpoint agent是如何优化的。

![此处输入图片的描述][1]


具体介绍请见[Pinpoint官方文档][2]

# pinpoint应用示例
这里给出一个例子关于如何从应用获取数据，这样就可以全面的理解前面讲述的内容。
![示例1：pinpoint应用][3]
下面阐述pinpoint为每个方法做了什么：

 - 当请求到达TomcatA时, Pinpoint agent 产生一个 TraceId.

> TX_ID: TomcatA\^TIME\^1  
SpanId: 10  
ParentSpanId: -1(Root)

 - 从spring MVC 控制器中记录数据
 - 插入HttpClient.execute()方法的调用并在HTTPGet中配置TraceId

> 创建一个子TraceId 
TX_ID: TomcatA\^TIME\^1 -> TomcatA\^TIME\^1 
SPAN_ID: 10 -> 20 
PARENT_SPAN_ID: -1 -> 10 (父 SpanId)
> 
> 在HTTP header中配置子 TraceId
> 
> HttpGet.setHeader(PINPOINT_TX_ID, "TomcatA\^TIME\^1")
> HttpGet.setHeader(PINPOINT_SPAN_ID, "20")
> HttpGet.setHeader(PINPOINT_PARENT_SPAN_ID, "10")

 - 传输打好tag的请求到TomcatB.

> TomcatB 检查传输过来的请求的header
> 
> HttpServletRequest.getHeader(PINPOINT_TX_ID)
> 
> TomcatB 作为子节点工作因为它识别了header中的TraceId
> 
> TX_ID: TomcatA\^TIME\^1 SPAN_ID: 20 PARENT_SPAN_ID: 10

 - 从spring mvc控制器中记录数据并完成请求
![此处输入图片的描述][4]
 - 当从tomcatB回来的请求完成时，pinpoint agent发送跟踪数据到pinpoint collector就此存储在HBase中
 - 在对tomcatB的HTTP调用结束后，TomcatA的请求也完成了。pinpoint agent发送跟踪数据到pinpoint
   collector就此存储在HBase中
 - UI从HBase中读取跟踪数据并通过排序树来创建调用栈

  
# 快速开始
注：内容翻译自 [官方quick start文档][5]，增加了少量补充和说明。

Pinpoint有三个主要组件(collector, web, agent)，并使用HBase作为存储。Collector和Web被打包为单个war文件，而agent被打包以便可以作为java agent附加到应用。

Pinpoint quickstart 为agent提供一个示例TestApp， 并使用tomcat maven插件来启动所有三个组件。

# 要求
为了构建pinpoint， 下列要求必须满足：

> 安装有JDK 6 
安装有JDK 8 
安装有Maven 3.2.x+ 
环境变量JAVA_6_HOME 设置为 JDK 6 home 目录
> 环境变量JAVA_7_HOME 设置为 JDK 7+ home 目录
环境变量JAVA_8_HOME 设置为 JDK 8+ home 目录

QuickStart 支持 Linux, OSX, 和 Windows.

注：没有说要不要安装jdk7，顺便一起安装吧。下面是/etc/profile的设置：

# use by pinpoint compile
export JAVA_6_HOME=/usr/lib/jvm/java-6-oracle/
export JAVA_7_HOME=/usr/lib/jvm/java-7-oracle/
export JAVA_8_HOME=/usr/lib/jvm/java-8-oracle/

# 开始
使用 git clone https://github.com/naver/pinpoint.git 下载pinpoint或者将项目作为zip文件打包下载然后解压。

使用maven安装pinpoint并运行 mvn install -Dmaven.test.skip=true

注:需要执行的命令如下：

> git clone https://github.com/naver/pinpoint.git 
cd pinpoint mvn
> install -Dmaven.test.skip=true



  [1]: http://paxblmm0h.bkt.clouddn.com/td_figure2.png
  [2]: https://github.com/naver/pinpoint/wiki/Technical-Overview-Of-Pinpoint
  [3]: http://paxblmm0h.bkt.clouddn.com/td_figure5.png
  [4]: http://paxblmm0h.bkt.clouddn.com/td_figure6.png
  [5]: https://github.com/naver/pinpoint/wiki/Technical-Overview-Of-Pinpoint