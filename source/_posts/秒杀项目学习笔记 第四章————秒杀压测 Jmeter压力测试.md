---
title: 秒杀项目学习笔记 第四章————秒杀压测 Jmeter压力测试
date: 2018-07-15 15:13:51
tags: [高并发,秒杀项目]
categories: 秒杀项目

---


# 秒杀项目学习笔记 第四章————秒杀压测 Jmeter压力测试


# 4-1 Jmeter入门
可以对网页进行压测

# 4-2 自定义变量

# 4-3  命令行压测
redis压测 **redis-benchmark**
1. redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000    100个并发 100000次访问
1.6s完成 100000次get  1s内完成79% 2s 96%    **6W/S**

2. redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100   存储大小为以100字节的数据包 

3.redis-benchmark -t set, lpush - q -n 100000  只测试set和lpush命令

4. redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
只执行此命令的压测

# Spring Boot 打war包  与jar包的区别？？？

1.添加spring-boot-starter-tomcat的provided依赖（编译时依赖，运行时不需要）
2.添加maven-war-plugin插件
3.修改 `<packaging>war</packaging>`
4 修改启动类,添加
```Java
public class MiaoshaApplication extends SpringBootServletInitializer{
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder){
    return builder.sources(MainApplication.class);
    }
}
```
5.到目录下  `mvn clean package` 与install有什么区别？？

# 服务器上压力测试
上传goods_list.jmx到服务器
`./apache-jmeter-3.3/bin/jmeter.sh -n -t goods_list.jmx -l result.jtl