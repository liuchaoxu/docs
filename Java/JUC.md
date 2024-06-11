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

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1677911125110-51bd36db-e553-4d5f-a4f3-088d96590b17.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_29%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

**方式二：通过 Executor 框架的工具类 Executors 来实现**

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1677911136796-4dad1cc4-f6fa-44a2-9e35-d293d9fb6fdb.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

根据《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 new ThreadPoolExecutor() 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险 , 所以我们在开发过程中创建线程池一般使用new ThreadPoolExecutor()手动设置参数的形式创建

# 线程池的7大核心参数

1. **核心线程数(corePoolSize) :** 定义了最小可以同时运行的线程数量。
2. **最大线程数(maximumPoolSize) :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
3. **工作队列(workQueue) :** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
4. **存活时间(keepAliveTime) :** 当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；
5. **时间单位(unit) :** keepAliveTime 参数的时间单位。
6. **线程工厂(threadFactory) :** executor 创建新线程的时候会用到。
7. **拒绝策略(handler) :** 关于饱和策略下面单独介绍一下
# 线程池任务调度流程
