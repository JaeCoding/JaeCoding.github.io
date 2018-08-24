---
title: lambda表达式的使用
date: 2018-05-24 11:35:26
tags: [Java]
categories: 编程技巧

---

# lambda表达式的使用
Lambda表达式是Java SE 8中一个重要的新特性

**作用：**
 1. 通过表达式来**代替功能接口**。
 2. 增强了集合库。Java SE 8添加了2个对集合数据进行批量操作的包: java.util.function
    包以及java.util.stream 包。 流(stream)就如同迭代器(iterator),但附加了许多额外的功能。

**包括：**正常的参数列表 和 一个使用这些参数的主体(body,可以是一个表达式或一个代码块)。



Lambda表达式的语法
基本语法:
`(parameters) -> expression`
或
`(parameters) ->{ statements; }`

**左边是参数，中间是->，右边是 表达式**

**整体是个返回值！！**

```Java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```