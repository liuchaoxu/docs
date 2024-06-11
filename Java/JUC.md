# 创建线程的方式


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

# 创建线程池


**方式一：通过构造方法实现**

根据传递的参数不同, 创建适用于不同场景的线程池


**方式二：通过 Executor 框架的工具类 Executors 来实现**

```java
List list = Arrays.asList("");
ExecutorService threadPool = Executors.newCachedThreadPool();
```

根据《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 new ThreadPoolExecutor() 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险 , 所以我们在开发过程中创建线程池一般使用new ThreadPoolExecutor()手动设置参数的形式创建

# 线程池的7大核心参数

1. **核心线程数(corePoolSize) :** 定义了最小可以同时运行的线程数量。
2. **最大线程数(maximumPoolSize) :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。即线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
3. **工作队列(workQueue) :** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
4. **存活时间(keepAliveTime) :** 当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；
5. **时间单位(unit) :** keepAliveTime 参数的时间单位。
6. **线程工厂(threadFactory) :** executor 创建新线程的时候会用到。表示生成线程池中工作线程的线程工厂， 用于创建线程，**一般默认的即可**
7. **拒绝策略(handler) :** 拒绝策略，表示当队列满了，并且工作线程大于 等于线程池的最大线程数（maximumPoolSize）时，如何来拒绝 请求执行的runnable的策略
# 线程池任务调度流程（线程池底层工作原理）
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
# 拒绝策略

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