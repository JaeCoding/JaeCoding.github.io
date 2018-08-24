---
title: Java生产环境下性能监控与调优
date: 2018-07-26 18:15:35
tags: [Java]
categories: 经验总结

---

# Java生产环境下性能监控与调优

用于生产环境
硬盘，网络，cpu问题

生产环境问题
特点：不好定位，不能重启，不能修改

 - 内存溢出如何处理
 - 给服务器分配多少内存
 - 对垃圾收集器 如何性能调优
 - CPU负载飙高
 - 应用分配多少线程
 - 不加log如何确定是否执行代码
 

 - JVM字节码
 - 字符串：添加，StringBUffer   i++与++i

 - 熟练使用各种工具监控和调试工具
 - 熟悉JVM字节码 

 - 学会自动内存回收机制，GC调优
 
# JVM参数类型

## 基于JDK 命令行工具 的监控

### JVM的参数类型
标准参数：-help -version 
**非标准参数：**

**X参数**
 2. ```-Xint```：完全 解释执行
 3. ```-Xcomp``` 第一次使用就 编译成本地代码（第一次速度较慢）
 4. ```-Xmixed```：混合模式，JVM自己决定是否 编译成本地代码   可在-version中查看


**XX参数**
**Boolean类型**
```-XX:[+]<name>```  +表示启用name属性，改成-就是禁用 比如```-XX:UseG1GC```启用G1垃圾器

 **非Boolean类型：**
```-XX:<name>=<value>```表示name属性的值为value
比如： ```-XX:MaxGCPauseMillis=500```GC停顿时间为500毫秒
-Xms等价于-XX:InitialHeapSize
-Xmx等价于-XX:MaxHeapSize

### 运行时JVM参数查看
-XX:+PrintFlagsInitial 查看初始值
-XX:+PrintFlagsFinal  查看最终值
-XX:+UnlockExperimentalVMOptions 解锁实验参数
-XX:+UnlockDiagnosticVMOptions 解锁诊断参数
-XX:+PrintCommandLineFlags 命令行参数

=表示默认值 :=表示被JVm修改后
测试
```java -XX:+PrintFlagsFinal -version > flags.txt
sz flags.txt 下载后查看```

**jinfo 使用**
http://blog.51cto.com/nolinux/1588716

**jstat 查看JVM统计信息** 注意 不是jstack
**查看类加载信息** -class option

**查看垃圾收集**-gc option 会显示堆内各分区内存容量如何
1.7的永久区变成了1.8的MetaSpace   你可以跟面试官说，是你通过jstat -gc 查看发现的
S0 S1， EDEN  OLD  MetaSpace  总量与使用量
CCS：压缩类空间（启用短指针的时候会出现）
YoungGC和OldGC的 次数 时间

[1.8移除PermGen][1]
堆区：和之前一样
非堆区：1.8移除了permGen，加入了Metaspace
Metaspace包括CCS，CodeCache（作用：开启JIT编译后，将Java代码转化为native代码，存放在CodeCache）

**查看JIT编译**
-compiler
多少个编译任务，失败个数，时间




Jstat查看虚拟机统计信息
Jmap+MAT实战内存溢出
jstack实战查看虚拟机统计信息
 
 


  [1]: https://blog.csdn.net/miaoao611/article/details/52677666