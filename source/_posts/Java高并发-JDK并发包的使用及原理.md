---
title: '[Java高并发]JDK并发包的使用及原理'
date: 2018-06-29 14:20:08
tags: [Java,高并发]
categories: 技术原理

---

[Java高并发]JDK并发包的使用及原理
# 一、同步控制工具
## ReentrantLock(可重入锁)
synchronized的特点：使用简单，一切交给JVM去处理，但是功能上是比较薄弱的。
在JDK1.5之前，ReentrantLock的性能要好于synchronized，由于对JVM进行了优化，现在的JDK版本中，两者性能是不相上下的。如果是简单的实现，不要刻意去使用ReentrantLock。

ReentrantLock的特点：在功能上更加丰富，它具有可重入、可中断、可限时、公平锁等特点。
```Java
public static ReentrantLock lock = new ReentrantLock();
	public static int i = 0;

	@Override
	public void run()
	{
		for (int j = 0; j < 10000000; j++)
		{
			lock.lock();
			try
			{
				i++;
			}
			finally
			{
				lock.unlock();
			}
		}
	}
```
ReentrantLock必须在finally中进行解锁操作，如果不在 finally解锁，有可能代码出现异常 锁没被释放，而synchronized是由JVM来释放锁。

### 特性一：可重入
单线程可以重复进入，但要重复退出
```Java
lock.lock();
lock.lock();
try{
	i++;
}			
finally{
	lock.unlock();
	lock.unlock();
}
```
可以反复得到相同的一把锁，它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放(重入锁)。这模仿了 synchronized 的语义；如果线程进入由线程**已经拥有的监控器保护**的 synchronized 块，就允许线程继续进行，当线程退出第二个（或者后续） synchronized 块的时候，不释放锁，只有线程退出它进入的监控器保护的第一个synchronized 块时，才释放锁。
### 特性二：可中断
与synchronized不同，ReentrantLock对中断是有响应的。

普通的lock.lock()是不能响应中断的，lock.lockInterruptibly()能够响应中断。
可用中断来处理死锁

### 特性三：可限时

超时不能获得锁，就返回false，不会永久等待构成死锁

使用`lock.tryLock(long timeout, TimeUnit unit)`来实现可限时锁，参数为时间和单位。

### 特性四：公平锁
使用方式：

```Java
public ReentrantLock(boolean fair) 

public static ReentrantLock fairLock = new ReentrantLock(true);
```
一般意义上的锁是不公平的，不一定先来的线程能先得到锁，后来的线程就后得到锁。不公平的锁可能会产生饥饿现象。

优点：保证线程是先来的先得到锁。不会产生饥饿现象
缺点：性能会比非公平锁差很多。

## Condition
Condition与ReentrantLock的关系就类似于synchronized与Object.wait()/signal()

await()方法会使当前线程等待，同时释放当前锁，当其他线程中使用signal()时或者signalAll()方法时，线程会重新获得锁并继续执行。或者当线程被中断时，也能跳出等待。这和Object.wait()方法很相似。

awaitUninterruptibly()方法与await()方法基本相同，但是它并不会再等待过程中响应中断。 singal()方法用于唤醒一个在等待中的线程。相对的singalAll()方法会唤醒所有在等待中的线程。这和Obejct.notify()方法很类似。
```Java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
public class Test implements Runnable
{
	public static ReentrantLock lock = new ReentrantLock();
	public static Condition condition = lock.newCondition();

	@Override
	public void run()
	{
		try
		{
			lock.lock();
			condition.await();//等待
			System.out.println("Thread is going on");
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
		finally
		{
			lock.unlock();
		}
	}
	
	public static void main(String[] args) throws InterruptedException
	{
		Test t = new Test();
		Thread thread = new Thread(t);
		thread.start();
		Thread.sleep(2000);
		
		lock.lock();
		condition.signal();//唤醒
		lock.unlock();
	}

}
```

## Semaphore （有额度的共享锁）
一般的锁，都是排他锁，是独占额。

而对于Semaphore（中文为信号，旗语）来说，它允许多个线程同时进入临界区。可以认为它是一个共享锁，但是共享的额度是有限制的，额度用完了，其他没有拿到额度的线程还是要阻塞在临界区外。当额度为1时，就相等于lock
```Java
public class Test implements Runnable
{
	final Semaphore semaphore = new Semaphore(5);
	@Override
	public void run()
	{
		try
		{
			semaphore.acquire();//线程去获取semaphore
			Thread.sleep(2000);
			System.out.println(Thread.currentThread().getId() + " done");
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}finally {
			semaphore.release();//释放
		}
	}
}
```
每个线程都去 Semaphore的许可，Semaphore的许可只有5个，运行后可以看到，5个一批，一批一批地输出。
## ReadWriteLock (读写锁)
ReadWriteLock是区分功能的锁。读和写是两种不同的功能，读-读不互斥，读-写互斥，写-写互斥。
```Java
private static ReentrantReadWriteLock readWriteLock=new ReentrantReadWriteLock(); //读写锁
private static Lock readLock = readWriteLock.readLock(); //从读写锁获取 读锁
private static Lock writeLock = readWriteLock.writeLock();//写锁
```
并且有非常著名的消费者生产者问题。

## CountDownLatch
倒数计时器
长用于全部线程执行计数导数后，在执行主线程

```Java
public class Test implements Runnable
{
	static final CountDownLatch countDownLatch = new CountDownLatch(10);
	static final Test t = new Test();
	@Override
	public void run()
	{
		try
		{
			Thread.sleep(2000);
			System.out.println("complete");
			countDownLatch.countDown();//减少
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) throws InterruptedException
	{
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		for (int i = 0; i < 10; i++)
		{
			executorService.execute(t);
		}
		countDownLatch.await();
		System.out.println("end");
		executorService.shutdown();
	}

}
```
## LockSupport

和suspend类似，提供线程阻塞原语

    LockSupport.park(); 
    LockSupport.unpark(t1);

与suspend相比 不容易引起线程冻结

LockSupport和Semaphore有点相似，内部有一个许可，park的时候拿掉这个许可，unpark的时候申请这个许可。所以如果unpark在park之前，是不会发生线程冻结的。
```
        @Override
        public void run()
        {
            synchronized (u)
            {
                System.out.println("in " + getName());
                //Thread.currentThread().suspend();
                LockSupport.park();
            }
        }
        
        LockSupport.unpark(t1);
```


# 二、ReentrantLock 的实现
ReentrantLock的实现主要由3部分组成：

 - CAS状态
 - 等待队列
 - park()

ReentrantLock的父类中会有一个state变量来表示同步的状态
```
/**
* The synchronization state.
*/
    private volatile int state;
```
通过`CAS`操作来设置state来获取锁，如果设置成了1，则将锁的持有者给当前线程
```Java
final void lock() {
            if (compareAndSetState(0, 1))//锁被设置成1则给当前
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);//否则申请
        }
```
如果拿锁不成功，则会做一个申请
```Java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))//把自己加到等待队列
            selfInterrupt();
    }
```
首先，再去申请下试试看tryAcquire，因为此时可能另一个线程已经释放了锁。
如果还是没有申请到锁，就addWaiter，意思是把自己加到等待队列中去
```Java
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
```
其间还会有多次尝试去申请锁，如果还是申请不到，就会被挂起
```Java
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```
同理，如果在unlock操作中，就是释放了锁，然后unpark

# 三、并发容器源码分析
## ConcurrentHashMap 
最简单的方式使HashMap变成线程安全就是使用Collections.synchronizedMap，它是对HashMap的一个包装,使用的是synchronized
```
public static Map m=Collections.synchronizedMap(new HashMap());
```
同理对于List，Set也提供了相似方法。
特点：适用于并发量比较小，高并发情况下性能堪忧
实现：将HashMap包装在里面，然后将HashMap的每个操作都加上synchronized。
由于每个方法都是获取同一把锁(mutex)，这就意味着，put和remove等操作是互斥的，大大减少了并发量。

而ConcurrentHashMap的实现
```Java
public V put(K key, V value) {
        Segment<K,V> s;
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }
```
相当于哈希表的哈希表。第一次hash得到一个HashMap的index，在此上再次hash得到确切的位置。
在 ConcurrentHashMap内部有一个Segment段，它将大的HashMap切分成若干个段（小的HashMap），然后让数据在每一段上Hash。

特点：多个线程在不同段上的Hash操作一定是线程安全的，所以只需要同步同一个段上的线程就可以了，这样实现了锁的分离，大大增加了并发量。

弊端：在使用ConcurrentHashMap.size时会比较麻烦，因为它要统计每个段的数据和，在这个时候，要把每一个段都加上锁，然后再做数据统计。这个就是把锁分离后的小小弊端，但是size方法应该是不会被高频率调用的方法。

在实现上，不使用synchronized和lock.lock而是尽量使用trylock，同时在HashMap的实现上，也做了一点优化。

## BlockingQueue
BlockingQueue不是一个高性能的容器。但是它是一个非常好的共享数据的容器。是典型的生产者和消费者问题的实现。
