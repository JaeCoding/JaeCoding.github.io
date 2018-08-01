---
title: java动态代理原理与实现
date: 2018-08-01 23:55:35
tags: [Java,动态代理,设计模式]
categories: 技术原理

---

# Java动态代理原理与实现

特征：代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。

代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。（有点像dubbo？） 

简单的说就是，我们在访问实际对象时，是通过代理对象来访问的，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。

## 静态代理

1、静态代理

由程序员创建或特定工具自动生成源代码，也就是在编译时就已经将接口，被代理类，代理类等确定下来。
在程序运行之前，代理类的.class文件就已经生成。

代理模式最主要的就是有一个公共接口（Person），一个具体的类（Student），一个代理类（StudentsProxy）,代理类持有具体类的实例，代为执行具体类实例方法。

假如一个班的同学要向老师交班费，但是都是通过班长把自己的钱转交给老师。这里，班长就是代理学生上交班费，

班长就是学生的代理。
```Java
public interface Person {
    void giveMoney();
}

public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    
    @Override
    public void giveMoney() {
       System.out.println(name + "上交班费50元");
    }
}
```
代理类(内置了被代理的对象）
StudentsProxy类，这个类也实现了Person接口，但是还另外**持有一个学生类对象**，由于实现了Peson接口，同时持有一个学生对象，那么他可以代理学生类对象执行上交班费（执行giveMoney()方法）行为。
```Java
public class StudentsProxy implements Person{
    //被代理的学生
    Student stu;
    
    public StudentsProxy(Person stu) {
        // 只代理学生对象
        if(stu.getClass() == Student.class) {
            this.stu = (Student)stu;
        }
    }
    
    //代理上交班费，调用被代理学生的上交班费行为
    public void giveMoney() {
        stu.giveMoney();
    }
}
```
那么，上交班费的时候，虽然操作的是代理类，而真正执行的是被代理的对象。
这里并没有直接通过张三（被代理对象）来执行上交班费的行为，而是通过班长（代理对象）来代理执行了。这就是代理模式。

上面说到，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。这里的间接性就是指不直接调用实际对象的方法，那么我们在代理过程中就可以加上一些其他用途。就这个例子来说，加入班长在帮张三上交班费之前想要先反映一下张三最近学习有很大进步，直接在班长的递交中加上即可。

可以看到，只需要在代理类中帮张三上交班费之前，执行其他操作就可以了。这种操作，也是使用代理模式的一个很大的优点。最直白的就是在**Spring中的面向切面编程（AOP），我们能在一个切点之前执行一些操作**，在一个切点之后执行一些操作，这个切点就是一个个方法。这些方法所在类肯定就是被代理了，在代理过程中切入了一些其他操作。

## 动态代理

### 1.动态代理

**定义：**代理类在程序**运行时创建的代理方式**被成为动态代理。

**对比：**静态代理的代理类(studentProxy)是自己定义好的，在程序运行之前就已经编译完成。
动态代理的代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的**“指示”动态生成**的。

**优势：**可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类中的方法。 

比如说，想要在每个代理的方法前都加上一个处理方法：
```Java
 public void giveMoney() {
        //调用被代理方法前加入处理方法,
        beforeMethod();
        stu.giveMoney();
}
```

这里只有一个giveMoney方法，就写一次beforeMethod方法，但是如果除了giveMonney还有很多其他的方法，那就需要写很多次beforeMethod方法，麻烦。那看看下面动态代理如何实现。

### 2、动态代理简单实现（`Proxy类`和`InvocationHandler接口`）

在java的`java.lang.reflect`包下提供了一个`Proxy类`和一个`InvocationHandler接口`，通过这个类和这个接口可以**生成JDK动态代理类**和**动态代理对象。**

 - 创建一个`InvocationHandler对象(调用处理器）`

```Java
//创建一个与 代理对象 相关联的InvocationHandler
InvocationHandler stuHandler = new MyInvocationHandler<Person>(stu);
```

 - 使用Proxy类的getProxyClass静态方法生成一个`动态代理类stuProxyClass类对象`（传入类加载器，类数组？）

```Java
Class<?> stuProxyClass = Proxy.getProxyClass(Person.class.getClassLoader(), new Class<?>[] {Person.class});
```
 - 获得stuProxyClass 中一个带InvocationHandler参数的`构造器constructor`
```Java
Constructor<?> constructor = PersonProxy.getConstructor(InvocationHandler.class);
```
 - 通过构造器constructor来创建一个`动态实例stuProxy`
```Java
Person stuProxy = (Person) cons.newInstance(stuHandler);
```
## 简化版本实现
就此，一个动态代理对象就创建完毕，当然，上面四个步骤可以通过Proxy类的newProxyInstances方法来简化：
```Java
 //创建一个与 代理对象 相关联的InvocationHandler (调用处理器）
InvocationHandler stuHandler = new MyInvocationHandler<Person>(stu);

//创建一个 代理对象stuProxy，代理对象的每个执行方法都会替换执行Invocation中的invoke方法
  Person stuProxy= (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler);
```
```Java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
```

> loader:一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载
> 
> interfaces:　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口(多个接口）给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了
> 
> h:　　一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上









## 举例实现

### 1.被代理对象
相同的例子：班长需要帮学生代交班费。
```Java
//被代理的接口
public interface Person {
    void giveMoney();
}
```

创建需要被代理的实际类：

```Java
public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    @Override
    public void giveMoney() {
        try {
          //假设数钱花了一秒时间
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
       System.out.println(name + "上交班费50元");
    }
}
```
再定义一个检测方法执行时间的工具类，在**任何方法执行前**先调用start方法，**执行后**调用finsh方法，就可以计算出该方法的运行时间，这也是一个最简单的方法执行时间检测工具。
```Java
public class MonitorUtil {
    //线程本地变量
    private static ThreadLocal<Long> tl = new ThreadLocal<>();
    
    public static void start() {
        tl.set(System.currentTimeMillis());
    }
    //结束时打印耗时
    public static void finish(String methodName) {
        long finishTime = System.currentTimeMillis();
        System.out.println(methodName + "方法耗时" + (finishTime - tl.get()) + "ms");
    }
}
```
### 2. 被代理对象 的 调用处理器
创建`StuInvocationHandler类`，实现InvocationHandler接口，这个类中持有一个**被代理对象的实例** target。InvocationHandler中有一个invoke方法，所有执行代理对象的方法都会被**替换成**执行invoke方法。

也就是说。本来我们想调用代理对象的XX方法，进而调用被代理对象的XX方法，但是实际执行的，是调用处理器的invoke()方法。

**invoke v 调用
Invocation n 调用**

再在invoke方法中执行被代理对象target的相应方法。当然，在代理过程中，我们在真正执行被代理对象的方法前加入自己其他处理。这也是**Spring中的AOP实现**的主要原理，这里还涉及到一个很重要的关于java反射方面的基础知识。

```Java
public class StuInvocationHandler<T> implements InvocationHandler {
   
   //invocationHandler持有的被代理对象
    T target;
    //此调用处理器 根据 被代理对象实例 构造
    public StuInvocationHandler(T target) {
       this.target = target;
    }
    
    /**
     * proxy:代表 动态代理对象 (班长大人）
     * method：代表 正在执行的方法 （缴费方法）
     * args：代表 调用目标方法时传入的实参  （这里缴费没用到参数）
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理执行" +method.getName() + "方法");
       
        //代理过程中插入监测方法,计算该方法耗时
        MonitorUtil.start();
        //    当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用 
        Object result = method.invoke(target, args);
        
        MonitorUtil.finish(method.getName());
        return result;
    }
}
```
### 3.创建动态代理对象
做完上面的工作后，我们就可以具体来创建动态代理对象了，上面简单介绍了如何创建动态代理对象，我们使用简化的方式创建动态代理对象

```Java
public class ProxyTest {
    public static void main(String[] args) {
        
        //创建 被代理对象 实例
        Person zhangsan = new Student("张三");
        
        //创建一个与 代理对象相关联的InvocationHandler
        InvocationHandler stuHandler = new StuInvocationHandler<Person>(zhangsan);
        
        //创建一个 代理对象stuProxy来代理zhangsan，代理对象 的每个执行方法都会替换执行Invocation中的 invoke方法
        //使用方法 Proxy.newProxyInstance 创建 代理对象实例
        //参数：1.类加载器 2.没注明长度的类对象数组 3.调用 
        Person stuProxy = (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler)；

       //代理执行上交班费的方法
        stuProxy.giveMoney();
    }
}
```

重点：Proxy.newProxyInstance的第二个参数。是
一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口。
看到两种选择，一种是`Person.getClass().getInterfaces()`
另一种是`new Class<?>[]{Person.class}`

最后执行的是StuInvocationHandler中的invoke方法

## 优势
很方便地对代理类的函数进行统一的处理，而不用修改每个代理类中的方法。是因为所有被代理执行的方法，都是通过在InvocationHandler中的invoke方法调用的，所以我们只要在invoke方法中统一处理，就可以对所有被代理的方法进行相同的操作了。

例如，我想给被代理对象加上 一个临时的记时功能。我不用去修改接口加上方法。 而是只要去Handler的invoke中加上。因为我们调用代理类的giveMoney()，实际上是调用过了Handler的invoke方法。

动态代理的过程中，我们并没有实际看到代理类，也没有很清晰地的看到代理类的具体样子，而且动态代理中被代理对象和代理对象是通过InvocationHandler来完成的代理过程的，其中具体是怎样操作的，为什么代理对象执行的方法都会通过InvocationHandler中的invoke方法来执行。带着这些问题，我们就需要对java动态代理的源码进行简要的分析，弄清楚其中缘由。

# 动态代理原理分析

## 1、Java动态代理创建出来的动态代理类

上面我们利用Proxy类的newProxyInstance方法创建了一个动态代理对象，查看该方法的源码，发现它只是封装了创建动态代理类的步骤(红色标准部分)：


查看newProxyInstance方法源码，可知


jdk为我们的生成了一个叫$Proxy0（这个名字后面的0是编号，有多个代理类会一次递增）的代理类，这个**类文件时放在内存中**的，我们在创建代理对象时，就是通过反射获得这个类的构造方法，然后创建的代理实例。通过对这个生成的代理类源码的查看，我们很容易能看出，动态代理实现的具体过程。

我们可以对**InvocationHandler看做一个中介类**，中介类持有一个**被代理对象**，在invoke方法中**调用了被代理对象的相应方法**。通过聚合方式持有被代理对象的引用，把外部对invoke的调用最终都转为对被代理对象的调用。

代理类调用自己方法时，通过**自身持有的中介类对象**来调用**中介类对象的invoke方法**，从而达到代理执行被代理对象的方法。也就是说，动态代理通过中介类实现了具体的代理功能。

通过 Proxy.newProxyInstance 创建的代理对象是在 jvm运行时动态生成的一个对象，它并不是我们的InvocationHandler类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象，并且命名方式都是这样的形式，以$开头，proxy为中，最后一个数字表示对象的标号。

