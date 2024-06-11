# ThreadPool线程池

## 线程池的优势

线程复用；控制最大并发数；管理线程。

1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
    
2. 提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行。
    
3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## 架构说明

Java中的线程池是通过Executor框架实现的，该框架中用到了**Executor，ExecutorService，ThreadPoolExecutor**这几个类。

Executor接口是顶层接口，只有一个execute方法，过于简单。通常不使用它，而是使用ExecutorService接口
## 创建线程的方式


1. 继承 Thread 类；
2. 实现 Runnable 接口；
3. 实现 Callable 接口；
4. 实现线程池创建
5. 使用匿名内部类方式
```java 
public class CreateRunnable {
    public static void main(String[] args) {
        //创建多线程创建开始
        Thread thread = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("i:" + i);
                }
            }
        });
        thread.start();
    }
}
```

## 创建线程池


**方式一：通过构造方法实现**

根据传递的参数不同, 创建适用于不同场景的线程池


**方式二：通过 Executor 框架的工具类 Executors 来实现**

```java
List list = Arrays.asList("");
ExecutorService threadPool = Executors.newCachedThreadPool();
```

根据《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 new ThreadPoolExecutor() 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险 , 所以我们在开发过程中创建线程池一般使用new ThreadPoolExecutor()手动设置参数的形式创建

## 线程池的7大核心参数

1. **核心线程数(corePoolSize) :** 定义了最小可以同时运行的线程数量。
2. **最大线程数(maximumPoolSize) :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。即线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
3. **工作队列(workQueue) :** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
4. **存活时间(keepAliveTime) :** 当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；
5. **时间单位(unit) :** keepAliveTime 参数的时间单位。
6. **线程工厂(threadFactory) :** executor 创建新线程的时候会用到。表示生成线程池中工作线程的线程工厂， 用于创建线程，**一般默认的即可**
7. **拒绝策略(handler) :** 拒绝策略，表示当队列满了，并且工作线程大于 等于线程池的最大线程数（maximumPoolSize）时，如何来拒绝 请求执行的runnable的策略
## 线程池任务调度流程（线程池底层工作原理）
提交任务 
--->核心线程池是否满了
（没有满，创建新线程执行任务）
---> 满了，判断任务队列池是否满了
（没有满，任务存放到队列中）
--->满了，判断最大线程数是否满了
（没有满，创建新的线程执行）
--->满了，将任务提交给**拒绝策略**处理

以下重要：

1. 在创建了线程池后，线程池中的**线程数为零**。
    
2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断：
    
    1. 如果正在运行的线程数量**小于corePoolSize**，那么马上**创建线程**运行这个任务；
        
    2. 如果正在运行的线程数量**大于或等于corePoolSize**，那么**将这个任务放入队列**；
        
    3. 如果这个时候队列满了且正在运行的线程数量还**小于maximumPoolSize**，那么还是要**创建非核心线程**立刻运行这个任务；
        
    4. 如果队列满了且正在运行的线程数量**大于或等于maximumPoolSize**，那么线程池会**启动饱和拒绝策略**来执行。
        
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
    
4. 当一个线程无事可做超过一定的时间（keepAliveTime）时，线程会判断： 如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉。 所以线程池的所有任务完成后，**它最终会收缩到corePoolSize的大小**。
## 拒绝策略

一般我们创建线程池时，为防止资源被耗尽，任务队列都会选择创建有界任务队列，但这种模式下如果出现**任务队列已满且线程池创建的线程数达到你设置的最大线程数时**，这时就需要你指定ThreadPoolExecutor的RejectedExecutionHandler参数即合理的拒绝策略，来处理线程池"超载"的情况。

ThreadPoolExecutor自带的拒绝策略如下：

1. AbortPolicy(默认)：直接**抛出RejectedExecutionException异常**阻止系统正常运行
    
2. CallerRunsPolicy：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是**将某些任务回退到调用者**，从而降低新任务的流量。
    
3. DiscardOldestPolicy：**抛弃队列中等待最久的任务**，然后把当前任务加人队列中 尝试再次提交当前任务。
    
4. DiscardPolicy：**该策略默默地丢弃无法处理的任务**，不予任何处理也不抛出异常。 如果允许任务丢失，这是最好的一种策略。


在实际项目中我们经常选择CallerRunsPolicy作为拒绝策略：

`CallerRunsPolicy` 的策略是，当线程池无法处理新任务时，不是抛弃任务，也不是抛出异常，而是将任务回退到提交任务的线程中去执行，这样做的目的是让提交任务的线程自己来执行这个任务，从而减轻线程池的压力。

使用 `CallerRunsPolicy` 可以在一定程度上避免任务丢失，并且让提交任务的线程参与任务的执行，这可能在某些场景下有助于避免系统过载。但是，这也会增加提交任务线程的负担，可能会影响到任务的提交性能。

在实际应用中，选择合适的拒绝策略需要根据具体的应用场景和系统设计来定。如果任务不能丢失，且系统能够承受提交线程处理任务的性能影响，`CallerRunsPolicy` 是一个不错的选择。

**以上内置的策略均实现了RejectedExecutionHandler接口，也可以自己扩展RejectedExecutionHandler接口，定义自己的拒绝策略**
简单示例：
```java
// 自定义连接池
ExecutorService threadPool = new ThreadPoolExecutor(
				2, 
				5,
                2,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                //new ThreadPoolExecutor.AbortPolicy()
                //new ThreadPoolExecutor.CallerRunsPolicy()
                //new ThreadPoolExecutor.DiscardOldestPolicy()
                //new ThreadPoolExecutor.DiscardPolicy()
		        new RejectedExecutionHandler() {
                    @Override
                    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                        System.out.println("自定义拒绝策略");
                    }
                }
        );
```

# 多线程高并发底层原理

## Java内存模型（JVM）

JMM规定了内存主要划分为**主内存**和**工作内存**两种。

> **主内存**：保存了所有的变量。 **共享变量**：如果一个变量被多个线程使用，那么这个变量会在每个线程的工作内存中保有一个副本，这种变量就是共享变量。 **工作内存**：每个线程都有自己的工作内存，线程独享，保存了线程用到的变量副本（主内存共享变量的一份拷贝）。工作内存负责与线程交互，也负责与主内存交互。

此处的主内存和工作内存跟JVM内存划分（堆、栈、方法区）是在不同的维度上进行的，如果非要对应起来，主内存对应的是Java堆中的对象实例部分，工作内存对应的是栈中的部分区域，从更底层的来说，**主内存对应的是硬件的物理内存，工作内存对应的是寄存器和高速缓存**。

JMM对共享内存的操作做出了如下两条规定：

- 线程对共享内存的所有操作都必须在自己的工作内存中进行，不能直接从主内存中读写；
    
- 不同线程无法直接访问其他线程工作内存中的变量，因此共享变量的值传递需要通过主内存完成。
    

内存模型的三大特性：

- **原子性：****即不可分割性。比如 a=0；（a非long和double类型） 这个操作是不可分割的，那么我们说这个操作是原子操作。再比如：a++； 这个操作实际是a = a + 1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要使用同步技术（sychronized）或者锁（Lock）来让它变成一个原子操作**。一个操作是原子操作，那么我们称它具有原子性。java的concurrent包下提供了一些原子类，我们可以通过阅读API来了解这些原子类的用法。比如：**AtomicInteger、AtomicLong、AtomicReference**等。
    
- **可见性：** 每个线程都有自己的工作内存，所以当某个线程修改完某个变量之后，在其他的线程中，未必能观察到该变量已经被修改。**在 Java 中 volatile、synchronized 和 final 实现可见性。** volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。
    
- **有序性：** java的有序性跟线程相关。一个线程内部所有操作都是有序的，如果是多个线程所有操作都是无序的。因为JMM的工作内存和主内存之间存在延迟，而且java会对一些指令进行重新排序。volatile和synchronized可以保证程序的有序性，很多程序员只理解这两个关键字的执行互斥，而没有很好的理解到volatile和synchronized也能保证指令不进行重排序。
    
    - volatile关键字本身就包含了禁止指令重排序的语义
        
    - synchronized则是由“一个变量在同一个时刻只允许一条线程对其进行lock操作”这条规则获得的，这个规则决定了持有同一个锁的两个同步块只能串行地进入

## volatile关键字（保证内存可见性）
验证volatile关键字保证内存可见性：
```java
public class VolatileDemo {

    private static [volatile] Integer flag = 1;

    public static void main(String[] args)  throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("我是子线程工作内存flag的值：" + flag);
                while(flag == 1){
                }
                System.out.println("子线程操作结束..." + flag);
            }
        }).start();
        Thread.sleep(500);
        flag = 2;
        System.out.println("我是主线程工作内存flag的值：" + flag);
    }
}
```
**没有添加**volatile关键字:
子线程读取不到主线程修改后的flag值，陷入死循环程序无法结束。
**添加**volatile关键字:
子线程可以读取的新值并结束子线程。

## CAS

CAS：Compare and Swap。比较并交换的意思。CAS操作有3个基本参数：内存地址A，旧值B，新值C。它的作用是将指定内存地址A的内容与所给的旧值B相比，如果相等，则将其内容替换为指令中提供的新值C；如果不等，则更新失败。类似于修改登陆密码的过程。当用户输入的原密码和数据库中存储的原密码相同，才可以将原密码更新为新密码，否则就不能更新。

**CAS是解决多线程并发安全问题的一种乐观锁算法。** 因为它在对共享变量更新之前，会先比较当前值是否与更新前的值一致，如果一致则更新，如果不一致则循环执行（称为自旋锁），直到当前值与更新前的值一致为止，才执行更新。

Unsafe类是CAS的核心类，提供**硬件级别的原子操作**（目前所有CPU基本都支持硬件级别的CAS操作）。

```java
// 对象、对象的属性地址偏移量、预期值、修改值1  
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

Unsafe简单demo：
```java

public class UnsafeDemo {  
​  
    private int number = 0;  
​  
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {  
        UnsafeDemo unsafeDemo = new UnsafeDemo();  
        System.out.println(unsafeDemo.number);// 修改前  
        unsafeDemo.compareAndSwap(0, 30);  
        System.out.println(unsafeDemo.number);// 修改后  
    }  
​  
    public void compareAndSwap(int oldValue, int newValue){  
        try {  
            // 通过反射获取Unsafe类中的theUnsafe对象  
            Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");  
            theUnsafe.setAccessible(true); // 设置为可见  
            Unsafe unsafe = (Unsafe) theUnsafe.get(null); // 获取Unsafe对象  
            // 获取number的偏移量  
            long offset = unsafe.objectFieldOffset(UnsafeDemo.class.getDeclaredField("number"));  
            // cas操作  
            unsafe.compareAndSwapInt(this, offset, oldValue, newValue);  
        } catch (NoSuchFieldException e) {  
            e.printStackTrace();  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        }  
    }  
}
```

### 基本代码演示

在JUC下有个atomic包，有很多原子操作的包装类。
这里以AtomicInteger这个类来演示：
```java

public class CasDemo {  
​  
    public static void main(String[] args) {  
        AtomicInteger i = new AtomicInteger(1);  
        System.out.println("第一次更新：" + i.compareAndSet(1, 200));  
        System.out.println("第一次更新后i的值：" + i.get());  
        System.out.println("第二次更新：" + i.compareAndSet(1, 300));  
        System.out.println("第二次更新后i的值：" + i.get());  
        System.out.println("第三次更新：" + i.compareAndSet(200, 300));  
        System.out.println("第三次更新后i的值：" + i.get());  
    }  
}
```

输出结果如下：
```cmd
第一次更新：true  
第一次更新后i的值：200  
第二次更新：false  
第二次更新后i的值：200  
第三次更新：true  
第三次更新后i的值：300
```


结果分析：

```cmd
第一次更新：i的值（1）和预期值（1）相同，所以执行了更新操作，把i的值更新为200  
第二次更新：i的值（200）和预期值（1）不同，所以不再执行更新操作  
第三次更新：i的值（200）和预期值（1）相同，所以执行了更新操作，把i的值更新为300
```
### 缺点

**开销大**：在并发量比较高的情况下，如果反复尝试更新某个变量，却又一直更新不成功，会给CPU带来较大的压力

**ABA问题**：当变量从A修改为B再修改回A时，变量值等于期望值A，但是无法判断是否修改，CAS操作在ABA修改后依然成功。

**不能保证代码块的原子性**：CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。