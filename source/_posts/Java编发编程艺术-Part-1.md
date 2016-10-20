---
title: Java编发编程艺术-Part 1
date: 2016-07-14 15:06:32
categories: Java
tags: 
    - Java
    - Concurrent Programming
---

### 并发编程的挑战

#### 多线程一定快吗 ？
**什么是上下文切换**：CPU通过分配给各个线程时间片进行线程间切换。在切换时，需要保持上一个任务的状态。任务从保存到加载的过程就是一次上下文切换。

**什么时候会慢**：当数据量不是太大，并发性不是那么好时，线程创建和上下文切换的开销将影响整体性能。

**如何减少上下文切换**：
- 无锁并发编程：通过较少并发程序中锁资源的争夺
- CAS算法：Compare and Set操作实现不用加锁。
- 使用最少线程：避免创建不需要的线程
- 携程：在单线程里实现多任务调度

**拓展**：
- **协程（Coroutine）**：
（待整理）
#### 死锁
**死锁发生情况**：当线程请求锁A后又去请求被占用锁B，而无法释放A；异常情况下无法释放锁；释放锁时出现异常。
**避免死锁**：
- 避免一个线程同时取得多个锁
- 避免一个线程在锁内占用多个资源 
- 尝试用定时锁，使用lock.tryLock(timeout)
- 对数据库锁，加锁和解锁必需在一个数据库连接里

### Java并发机制底层实现原理
#### volatile 
**定义**：
Java编程语言允许线程访问共享变量，必需使用volatile来修饰该变量，已达到准确和一致性。
**原理**：
Step 1：volatile变量修饰的共享变量进行写操作时，编译器生成汇编代码前会加LOCK前缀
Step 2：当前处理器执行这条LOCK指令时，会锁住缓存（总线），将该缓存行数据写入内存。
Step3 ：其他处理器通过总线嗅探检查各自的缓存是否过期。如果过期，则将缓存行置为无效状态。则下次读该数据时，会强行从内存中读取。
![Alt text](./1447251879895.png)


**拓展**
- **缓存行优化**：
在一个队列中，原始方案头部结点的引用和尾部结点的引用都是四个字节，在同一个缓存行中。如果对头结点进行修改，将锁定该缓存行，使其它线程不能修改为尾部结点的引用。因此可以扩展头部结点的方法，使其不在同一个缓存行。
```java
/** head of the queue */
private transient final PaddedAtomicReference<QNode> head;
/** tail of the queue */
private transient final PaddedAtomicReference<QNode> tail;
static final class PaddedAtomicReference <T> extends AtomicReference <T> {
  // enough padding for 64bytes with 4byte refs
  Object p0, p1, p2, p3, p4, p5, p6, p7, p8, p9, pa, pb, pc, pd, pe;
  PaddedAtomicReference(T r) {
    super(r);
  }
}

public class AtomicReference <V> implements java.io.Serializable {
  private volatile V value;
  //省略其他代码
｝
```
优化的代码中，利用15个成员变量拓展了优化的头结点，使其包含64个字节，这样对head进行修改的时就不会影响tail变量。
*参考链接*：
[伪共享](http://ifeve.com/falsesharing/)
- **increase操作的原子性**：
```java
public class Test {
    public volatile int inc = 0; 
    public void increase() {
        inc++;
    }
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        } 
	    while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```
上述操作并不能保证increase操作的原子性，因为inc++操作不是原子性的。它分为三个步骤，读取，累加，写入。有可能线程1读取后，被线程2读取并写回，此时线程1的数据为脏数据。
*参考链接*
*[Volatile 关键字解析](http://www.cnblogs.com/dolphin0520/p/3920373.html)*

#### synchroized
**定义**：
Java中每个对象都可以作为锁，具体表现为：
- 对于普通同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前类的class的对象
- 对于同步方法块，锁是synchonizd括号里配置的代码块。

**原理**：
在JVM中，利用了进入和退出Monitor对象来实现方法同步和代码块同步。但实现细节不一样，代码块同步使用的是monitorenter和moniterexit指令实现的。任何一个对象都有一个monitor与之关联，当一个monitor被持有后，它将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor的所有权。
```java
synchronized{
	代码块
	....
}
-->
monitorenter;
指令
....
monitorexit;
```
**拓展**：
- **Monitor详解**
(待整理)
#### Java对象头
synchronized用的锁状态存在Java对象头中，如果对象是数组类型，则使用3个字（Word）存储对象头，否则则使用2个字存储。
![Alt text](./1447314021987.png)

在MarkWork中会标记当前对象所在状态：
![Alt text](./1447313985294.png)

####  锁的升级与比较
1、偏向锁
当一个线程访问同步块并获取锁时，会在对象头和栈帧的锁记录里存储线程ID。以后再进入或退出锁时，只需要简单地测试对象头里是否存储着指向当前线程的偏向锁。如果成功，则说明线程获得了锁；如果失败，则测试是否为偏向锁：如果没有，则使用CAS获得锁，如果设置了，则使用CAS将偏向锁指向当前线程（抢锁）。
偏向锁的撤销：当其它线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁或者升级为轻量级锁。而且，必需要等待全局安全点（没有正在执行的字节码）。
**优点**：减少获得锁和释放锁带来的性能消耗，使长期使用一个锁的线程不需要竞争锁。
![Alt text](./1447317788024.png)

**拓展**：
- **JVM虚拟机启动时偏向锁的优化**
在JVM中可以使用*-XX:+UsedBiasedLocking*来启用偏向锁，JDK1.6是默认使用偏向锁。如果在确认程序不会发生线程竞争的情况下，也可以使用*-XX:BiasedLockingStartupDelay=0*来让JVM启动时就使用偏向锁。实验如下：
```java
class SynchronizeCounter {
    private static long count = 0l;
    public synchronized void increment() {
        count = count + 1;
    }
    public long getCount() {
        return count;
    }
}
public class BiasedLockTest {
    public static void main(String[] args) throws InterruptedException {
        SynchronizeCounter counter = new SynchronizeCounter();
        long tsStart = System.currentTimeMillis();
        for(int i = 0;i < 100000000;i++){
            counter.increment();
        }
        System.out.println("累加"+counter.getCount()+"一共耗费："
                + (System.currentTimeMillis() - tsStart) + " ms");
    }
}
```
实验结果如下：
| 使用参数|     时间|   备注|
| :-------- | --------:| :------: |
| 无|   509 ms|  不使用任何参数|
| **-XX:BiasedLockingStartupDelay=0**|   69 ms|  立即使用偏向锁|
| **-XX:-UsedBiasedLocking**|501 ms|不使用偏向锁|
**分析**：因为默认JVM虚拟器开启4s后才使用偏向锁，而此例运行时间较短，因此不能显示禁用BiasedLocking的劣势。

2、轻量级锁：
加锁：执行同步块之前，在线程的栈帧中创建用于存储锁记录的空间，并将对象头的Mark Word复制到锁记录中。然后尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。
解锁：使用CAS操作将Mark Word替换回对象头。如果成功，则没有竞争发生。如果失败，则当前锁存在竞争，锁会膨胀为重量级锁。
![Alt text](./1447315953171.png)
注意：当其它线程在尝试获得已被加锁的轻量级锁时，会先自旋N次（N由JVM选项控制），如果在这N次自旋的过程中获得了锁，那说明竞争不激烈。如果没有获得，则将锁升级为重量级锁，然后将自己挂起。

![Alt text](./1453082565611.png)
初始状态下的线程执行栈帧和对象头

![Alt text](./1453082588282.png)
轻量级锁使用CAS操作复制MardWord和将对象头里的指针置为栈帧中的地址的示意。

3、重量级锁
重量级锁即操作系统内核态互斥锁，使用时需切换到内核态，开销巨大。

4、锁的优缺点（表格）
![Alt text](./1447318490264.png)

**拓展**
- **Java 锁机制**
（待整理）

#### 原子操作的实现原理
1、处理器如何实现原子操作
- 使用总线锁保证原子性：锁住总线
- 使用缓存锁保证原子性：锁住缓存行

2、Java如何实现原子操作
在Java中可以通过锁和循环CAS方法来实现原子操作
**CAS操作**：
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Created by THU on 2015/11/2.
 */
public class Counter {
    private AtomicInteger atomicI = new AtomicInteger(0);
    private int i = 0;

    /**
     * 使用CAS操作，线程安全的计数器
     */
    private void safeCount(){
        for(;;){
            int i = atomicI.get();
            boolean suc = atomicI.compareAndSet(i,++i);
            if(suc){
                break;
            }
        }
    }

    /**
     * 非线程安全的计数器
     */
    private void count(){
        i++;
    }

    public static void main(String[] args){
        final Counter cas = new Counter();
        List<Thread> ts = new ArrayList<Thread>(200);
        long start = System.currentTimeMillis();
        for(int j=0;j<100;j++){
            Thread t = new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i = 0;i<10000;i++){
                        cas.count();
                        cas.safeCount();
                    }
                }
            });
            ts.add(t);
        }
        for (Thread t : ts){
            t.start();
        }

        //等待所有线程执行完成
        for(Thread t:ts){
            try{
                t.join();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }

        System.out.println(cas.i);
        System.out.println(cas.atomicI.get());
        System.out.println(System.currentTimeMillis()-start);
    }
}
```
CAS操作通过调用JNI（Java Native Inerface)来获取底层汇编的支持，利用调用C++的接口。
C++源码里面会对是否为多处理器进行判断，然后判断是否加上#LOCK前缀。
CAS问题：
- ABA问题：如果一个值原来是A，变化成了B，又返回成A值，一种解决办法是利用版本号。JDK 1.5中利用AtomicStampedReference来进行解决。
- 如果自旋CAS操作长时间不成功，会给CPU带来很大的开销。
- 只能保证一个共享变量的的原子操作，可以采用将两个变量合起来的方法，也可以将几个变量合并在一个对象里面。使用AtomicReference类来保证引用对象的原子性。

*参考链接*：
*[JAVA并发编程学习笔记之CAS操作](http://blog.csdn.net/hsuxu/article/details/9467651)*
*[用AtomicStampedReference解决ABA问题](http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html)*



