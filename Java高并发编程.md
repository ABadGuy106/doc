# Java高并发编程

## 基本概念

### 并发：

同时拥有两个或者多个线程，如果程序在单核处理器上运行，多个线程将交替地换入或者换出内存，这些线程是同时"存在"的，每个线程都处于执行过程中的某个状态，如果运行在多核处理器上，此时，程序中的每个线程都将分配到一个处理器核上，因此可以同时运行

### 高并发：

是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计保证系统能够同时并行处理很多请求

## 并发编程基础



## 线程安全性

#### 定义：

当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些进程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类时线程安全的。

#### 线程安全性：

原子性：提供了互斥访问，同一时刻只能有一个线程来对它进行操作

​	涉及到jdk的  Atomic包

##### AtomicXXX: CAS、Unsafe.compareAndSwapInt

```java
count.incrementAndGet();

public class AtomicInteger extends Number implements java.io.Serializable {
  /**
     * Atomically increments by one the current value.
     *
     * @return the updated value
     */
    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
}


public final class Unsafe {
  
  	/**
  	*  var1  当前对象，即调用对象 count
  	*  var2  当前值，例如2+1  当前值为  2
  	*  var4  增加量
  	**/
      public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
  
}
```

​	（2）AtomicLong、LongAdder

| AtomicLong                              | LongAdder                    |
| --------------------------------------- | ---------------------------- |
|                                         | 在统计的时候如果有并发更新，可能会导致统计的数据有误差  |
| 如果在序列号生成或者要求很精确的情况下，必须使用全局唯一的AtomicLong | 高并发性能比AtomicLong强            |
| 在线程竞争很低的情况使用AtomicLong更简单，效率更高一些        | 在实际处理中在高并发状态下可以优先使用LongAdder |

 ![atomic包下的类](image/atomic.PNG)

AtomicReference、AtomicReferenceFieldUpdater

##### AtomicStampReference: CAS的ABA问题

ABA问题：

​	当一个线程要操作某个变量时，首先拿到该变量值为A当这个线程对该变量进行操作时，另一个线程拿到该变量并将该变量值修改为B后，又修改回了A。这时第一个线程回来发现这个变量的值还是A没有改变，所以做了其他操作。

​	解决办法：当第一个线程拿到该变量时为其设置版本号1，第二个线程将变量A改为B时，变量版本号变为2，当第二个线程将变量由B改为A的时候，变量版本变为3。当第一个线程回来时发现版本号不是1则表示变量已经被修改

```java
public class AtomicStampedReference<V> {
  /**
     * Atomically sets the value of both the reference and stamp
     * to the given update values if the
     * current reference is {@code ==} to the expected reference
     * and the current stamp is equal to the expected stamp.
     *
     * @param expectedReference the expected value of the reference
     * @param newReference the new value for the reference
     * @param expectedStamp the expected value of the stamp
     * @param newStamp the new value for the stamp
     * @return {@code true} if successful
     */
    public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
}
```

##### 原子性-锁

​	synchronized: 依赖JVM    同步锁

​		synchronized的对象主要由四种：

​			修饰代码块（同步语句块）：大括号括起来的代码，作用于调用的对象

​			修饰方法（同步方法）：整个方法，作用于调用的对象

​			修饰静态方法：整个静态方法，作用于所有对象

​			修饰类：括号括起来的部分，作用于所有对象

​		ps:如果某个类的方法用syschronized关键字修饰，另一个类继承该类的时候，不继承该方法的syschronized关键	字。	因为synchronized不属于方法声明的一部分。如果子类要使用syschronized需要自己显示的在方法上声明synchronized。

​	synchronized:不可中断锁，适合竞争不激烈，可读性好

​	Lock:可中断锁，多样化同步，竞争激烈时能维持常态

​	Atomic:竞争激烈时能维持常态，比Lock性能好；只能同步一个值

##### Lock : 依赖特殊的CPU指令，代码实现，ReentrantLock

#### 可见性：一个线程对主内存的修改可以及时的被其他线程观察到

导致共享变量在线程间不可见的原因：

- 线程交叉执行
- 重排序结合线程交叉执行
- 共享变量更新后的值没有在工作内存于主存间及时更新

重排序

#### 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般杂乱无序







## 安全发布对象

- 发布对象：使一个对象能够被当前范围之外的代码所使用
- 对象逸出：一种错误的发布。当一个对象还没有构造完成时，就使它被其他线程所见

1. #### 安全发布对象的４种方式：

2. 在静态初始化函数中初始化一个对象引用

3. 将对象的引用保存到volatile类型或者AtomicReference对象中

4. 将对象的应用保存到抱够正确构造对象的final类型域中

5. 将对象的引用保存到一个由锁保护的域中

## 线程安全策略

#### 不可变对象（不可变对象参考类　java.lang.String）

不可变对象需要满足的条件：

- ​	对象创建以后其状态就不能修改
  - ​对象所有于都是final类型
  - ​对象是正确创建的（在对象创建期间，this引用没有逸出）

将集合转换为不可修改集合

Collections.unmodifiableXXX : Collection、List、Set、Map ...

Guava: ImmutableXXX : Collection、List、Set 、Map ...

#### 线程封闭

Ad-hoc 线程封闭：程序控制实现，最糟糕，忽略

堆栈封闭：局部变量，无并发问题

ThreadLocal 线程封闭：特别好的封闭方法

#### 常见线程不安全类与写法

StringBuilder -> StringBuffer

SimpleDateFormat -> JodaTime

ArrayList、HashSet、HashMap等Collections

先检查再执行：if(condition(a)){handle(a);}

#### 同步容器

ArrayList 	->	 Vector,Stack

HashMap	->	HashTable(key,value不能为null)

Collections.synchronizedXXX(List、Set、Map)







































































































































































