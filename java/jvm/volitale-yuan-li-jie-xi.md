# Volitale原理解析



## Volitale

在并发编程中，我们必须要考虑3个问题：

* 可见性\(内存副本和主内存\)
* 原子性\(\)
* 有序性\(禁止重排序\)

`Synchronized`可以保证这3点，但是比较重.

`Volitale`可以保证保证：可见性，有序性.

{% hint style="danger" %}
对于原子性，需要强调一点，也是大家容易误解的一点：对volatile变量的单次读/写操作可以保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。
{% endhint %}

### Volatile缓存可见性实现原理

底层实现主要是通过**汇编lock前缀**指令, 它会锁定这块内存区域的缓存(**缓存行锁定**)并回写到主内存

IA-32和Intel 64架构软件开发者手册对lock指令的解释:

1. 会将当前处理器缓存行的数据**立即**写会到系统内存.
2. 这个写回内存操作会引起在其他CPU缓存了改内存地址数据无效(MESI协议)
3. 提供内存屏障功能,使lock前后指令**不能重排序**

### 指令重排序

遵守2个原则: `as-if-serial` 和`happens-before`

```java
public class TestDCL {
    private static volatile TestDCL sInstance;

    private TestDCL(){}

    public static TestDCL getInstance(){
        if(sInstance == null){
            synchronized(TestDCL.class){
                if(sInstance == null){
                    sInstance = new TestDCL();
                }
            }
        }
        return sInstance;
    }
}
```

```
// class version 52.0 (52)
// access flags 0x21
public class TestDCL {

  // compiled from: TestDCL.java

  // access flags 0x4A
  private static volatile LTestDCL; sInstance

  // access flags 0x2
  private <init>()V
   L0
    LINENUMBER 11 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this LTestDCL; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x9
  public static getInstance()LTestDCL;
    TRYCATCHBLOCK L0 L1 L2 null
    TRYCATCHBLOCK L2 L3 L2 null
   L4
    LINENUMBER 14 L4
    GETSTATIC TestDCL.sInstance : LTestDCL;
    IFNONNULL L5
   L6
    LINENUMBER 15 L6
    LDC LTestDCL;.class
    DUP
    ASTORE 0
    MONITORENTER
   L0
    LINENUMBER 16 L0
    GETSTATIC TestDCL.sInstance : LTestDCL;
    IFNONNULL L7
   L8
    LINENUMBER 17 L8
    NEW TestDCL
    DUP
    INVOKESPECIAL TestDCL.<init> ()V
    PUTSTATIC TestDCL.sInstance : LTestDCL;
   L7
    LINENUMBER 19 L7
   FRAME APPEND [java/lang/Object]
    ALOAD 0
    MONITOREXIT
   L1
    GOTO L5
   L2
   FRAME SAME1 java/lang/Throwable
    ASTORE 1
    ALOAD 0
    MONITOREXIT
   L3
    ALOAD 1
    ATHROW
   L5
    LINENUMBER 21 L5
   FRAME CHOP 1
    GETSTATIC TestDCL.sInstance : LTestDCL;
    ARETURN
    MAXSTACK = 2
    MAXLOCALS = 2
}


// 如果没有volatile关键字,那么这里可能发生指令重排序
// 1, 2可能换位置
INVOKESPECIAL TestDCL.<init> ()V  // 1
PUTSTATIC TestDCL.sInstance : LTestDCL; // 2
```

### 内存屏障相关

* JVM内存屏障规范

  * LoadLoad
  * StoreStore
  * LoadStore
  * StoreLoad

* CPU硬件对JVM内存屏障的实现

  Intel CPU实现

  > * Ifence: 是一种Local Barrier读屏障, 实现LoadLoad屏障
  > * sfence: 是一种store Barrier写屏障, 实现StoreStore屏障
  > * mfence: 是一种全能型的屏障, 具备Ifence和sfence的能力, 具有所有屏障功能

  JVM底层简化了内存屏障硬件指令的实现

  * lock前缀: lock指令不是一种内存屏障, 但是它能实现类似内存屏障的功能.


***

## synchronized 

synchronized 内置锁 是一种 对象锁（锁的是对象而非引用变量），**作用粒度是对象 ，可以用来实现对 临界资源的同步互斥访问 ，是 可重入 的。其可重入最大的作用是避免死锁**.

> **子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁；**

**synchronized给出的答案是在软件层面依赖JVM，而j.u.c.Lock给出的答案是在硬件层面依赖特殊的CPU指令。**



* 锁住对象

  ```java
  package com.paddx.test.concurrent;
  public class SynchronizedDemo {
      public void method() {
          synchronized (this) {
              System.out.println("Method 1 start");
          }
      }
  }
  
  // ----------------------字节码层面
  monitorenter, monitorexit
  // ----------------------------------操作系统层面
  mutex
  ```

  > 1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；
  >
  > 2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1；
  >
  > 3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权
  >
  >
  >    monitorexit指令出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁；

* 锁住方法

  ```java
  package com.paddx.test.concurrent;
  public class SynchronizedMethod {
      public synchronized void method() {
          System.out.println("Hello World!");
      }
  }
  
  // ----------------------字节码层面
  ACC_SYNCHRONIZED 标示符
  // ----------------------------------操作系统层面
  mutex
  ```

两**个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现**



[很好的一篇解释文章](https://www.cnblogs.com/aspirant/p/11470858.html)

## 监视器（Monitor）

Q: 什么是Monitor

A: 抽象的定义, 具体到代码里面是通过`ObjectMonitor`对象跟一系列操作来实现的.

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

