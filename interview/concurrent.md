## 0x00. start()和run()

1. start()和run()都是Thread类的方法（如果用的是Runnable则执行的是Runnable中的run方法，注意Callable中的是call方法）
2. start是启动线程作用是将线程变为就绪状态，至于是否调用还是得看CPU的分配。run是一个线程的具体执行内容，线程启动后自动调用。
3. 如果在main线程中调用了t1线程的run方法，就相当于main线程运行了一个普通的run方法，并没能达到多线程的效果
4. strat只能调用一次，多次调用会抛`IllegalThreadStateException`异常

## 0x01. sleep()与yield()

**sleep**

* 调用sleep会让当前线程从`Running`进入`Timed Waiting`（阻塞状态）状态
* 其他线程可以使用`interrupt`方法打断正在睡眠的线程，这时sleep方法会抛出`InterruptedException`
* 睡眠结束后的线程未必会立刻得到执行
* 建议用TimeUnit的sleep代替Thread的sleep来获得更好的可读性

**yield**

* 调用yield会让当前线程从`Running`进入`Runnalbe`（就绪状态）状态，然后调度执行其他同优先级的线程。如果这时没有同优先级的线程，那么不能保证让当前线程暂停的效果
* 具体的实现依赖于操作系统的任务调度器

**区别**

就绪状态有机会被任务调度器调用，阻塞状态不会。

sleep有休眠时间，yield没有时间参数

## 0x02. interrupt()

> Thread中的方法

* 如果打断的是阻塞线程(`sleep`, `wait`, `join`)，则打断标记(`isInterrupted()`)会在打断后清为False
* 如果打断的是正常运行的线程，则不会清空打断状态

## 0x03. 线程优先级

Java中优先级最大10，最小1，默认为5，仅仅是一个提示，调度器甚至可以忽略。

只有在cpu比较忙的时候，优先级较高的线程会获得更多的时间片，cpu空闲时，优先级几乎没什么用。

## 0x04. 两阶段终止模式

在线程T1中如何优雅地终止另一个线程T2？这里的优雅指的是给T2一个处理其他事情的机会（如释放锁）

如果调用线程的`stop()`方法，如果此时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其他线程永远无法获取锁。

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/889b421e38b1d734bb96cbf20feb4664.png)

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Monitor monitor = new Monitor();
        monitor.start();
        Thread.sleep(3500);
        monitor.stop();
    }
}

class Monitor {

    Thread monitor;

    /**
     * 启动监控器线程
     */
    public void start() {
        //设置线控器线程，用于监控线程状态
        monitor = new Thread() {
            @Override
            public void run() {
                //开始不停的监控
                while (true) {
                    //判断当前线程是否被打断了
                    if(Thread.currentThread().isInterrupted()) {
                        System.out.println("处理后续任务");
                        //终止线程执行
                        break;
                    }
                    System.out.println("监控器运行中...");
                    try {
                        //线程休眠
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        //如果是在休眠的时候被打断，不会将打断标记设置为true，这时要重新设置打断标记
                        Thread.currentThread().interrupt();
                    }
                }
            }
        };
        monitor.start();
    }

    /**
     * 	用于停止监控器线程
     */
    public void stop() {
        //打断线程
        monitor.interrupt();
    }
}
```

## 0x05. 守护线程

当Java进程中有多个线程执行时，只有当所有非守护线程执行完毕后，Java进程才会结束。但当非守护线程执行完毕后，守护线程无论是否执行完毕，都会一同结束。

> 垃圾回收器就是一个守护线程

## 0x06. 线程状态

**五种状态**

> 操作系统层面

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220225202048787.png)

* 初始状态：仅在语言层面创建了线程对象，还未与操作系统线程关联
* 可运行状态（就绪状态）：指该线程已经被创建（与操作系统线程相关），可以由CPU调度使用
* 运行状态：指获取了CPU时间片运行中的状态
* 阻塞状态：
  * 如果调用了阻塞API，如读写文件，这时该线程实际不会用到CPU，会导致线程上下文切换，进入阻塞状态
  * 等读写完毕，会由操作系统唤醒阻塞的线程，转换至可运行状态
  * 与**可运行状态**的区别是，对阻塞状态的线程来说只要他们一直不唤醒，调度器就一直不会考虑调度他们。
* 终止状态：表示线程执行已经完毕，生命周期已经结束，不会再转换为其他状态

**六种状态**

> Java中Thread.State枚举描述的

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220225202815248.png)

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220303171324316.png)

* NEW：线程刚被创建，但是还没有调用start()方法
* RUNNABLE：当调用了start()方法之后的状态。涵盖了操作系统层面的【可运行状态】、【运行状态】和【阻塞状态】（在Java中无法区分运行状态和可运行状态）
* BLOCKED、WAITING、TIMED_WAITING：都是Java API层面对【阻塞状态】的细分
* TERMINATED：当前线程运行结束

![Java 线程的状态 ](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

## 0x07. 变量的线程安全分析

**成员变量和静态变量的线程安全分析**

* 如果变量没有在线程间共享，那么变量是安全的
* 如果变量在线程中共享
  * 如果只有读操作，则线程安全
  * 如果有写操作，则该变量属于临界资源，需要考虑线程安全问题

**局部变量线程安全分析** 

* 局部变量被初始化为基本数据类型则是安全的
* 当局部变量是引用变量时则需要进行逃逸分析判断
  * 如果该对象没有逃离方法的作用范围，则线程安全
  * 如果该对象逃离了方法的作用范围，则线程不安全

## 0x08. 对象头

Java对象头详细信息在JVM中有描述，简要来说包含`Mark Word`(32bit)和`Klass Word`(32bit)。如果是数组的话还会包含数组长度(32bit)。

下图描述的是不同锁状态下Mark Word的形式，其中后几位为001表示无锁，101表示偏向锁，00表示轻量级锁，10表示重量级锁，11表示标记GC

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/0ffaeb7ddf7d71801bfd3eeb00754162.png)

## 0x09. Monitor原理

Monitor被翻译成**监视器**或**管程**

每个Java对象都可以关联一个Monitor对象，如果使用synchronized给对象上锁之后，该对象头的Mark Word中就被设置成指向Monitor对象的指针

Monitor的结构如下：

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220227152703623.png)

* 刚开始Monitor中的Owner为null
* 当Thread-2执行synchronized(obj)就会将Monitor的所有者Owner置为Thread-2，Monitor中只能有一个Owner
* 在Thread-2上锁的过程中，如果Thread-3，Thread-4，Thread-5也来执行synchronized(obj)，就会进入EntryList BLOCKED
* Thread-2执行完同步代码块的内容，然后唤醒EntryList中等待的线程来竞争锁，竞争的时候是非公平的
* 途中WaitSet中的Thread-0，Thread-1是之前获得过锁，但条件不满足进入WAITING状态的线程

注意：

* synchronized必须是进入同一个锁对象的monitor才有上述的效果（一个锁对象对应着一个monitor）
* 不加synchronized的对象不会关联监视器，不遵从上述规则

**字节码层面分析synchronized**

* monitorenter是进入synchronized语句
* monitorexit是退出synchronized语句
* 6 - 14行是synchronized中执行的部分，如果其中出现了错误也会释放锁，因为异常表中当在6 - 16行出现异常时，会跳到19行执行异常处理部分。

```java
static final Object lock = new Object();
static int counter = 0;
public static void main(String[] args) {
    synchronized (lock) {
        counter++;
    }
}
```

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/20201219201521709.png)

## 0x0A. 自旋优化

> 优化重量级锁竞争

当发生**重量级锁竞争**的时候，还可以使用自旋来进行优化（不加入Monitor的阻塞队列EntryList中），如果当前线程自旋成功（即在自旋的时候持锁的线程释放了锁），那么当前线程就可以不用进行上下文切换（持锁线程执行完synchronized同步块后，释放锁，Owner为空，唤醒阻塞队列来竞争，胜出的线程获取cpu执行权的过程）就获得了锁

**成功演示：**

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/39ed180b2ab7eae1bc37ebba0a819c4c.png)

**失败演示：**

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/36162c78749df99fcd83560e3896aef0.png)

自旋会占用CPU时间，单核CPU自选就是浪费，多核CPU自旋才能发挥优势

## 0x0B. 轻量级锁

> 用于优化重量级锁
>
> https://blog.csdn.net/m0_37989980/article/details/111408759#t5

## 0x0C. 偏向锁

> 用于优化轻量级锁重入
>
> https://blog.csdn.net/m0_37989980/article/details/111408759#t8

## 0x0D. sleep()和wait()的区别

1. sleep是Thread方法，wait是Object方法
2. sleep不需要强制和synchronized配合使用，但wait需要和synchronized一起使用
3. **sleep不会释放锁对象，wait会释放锁对象**

他们的线程状态都是`TIMED_WAITING`

## 0x0F. 保护性暂停模式

> 用于一个线程等待另一个线程的执行结果

* 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个GuardedObject
* 如果有结果不断从一个线程到另一个线程，那么可以使用消息队列（生产者消费者模式）
* JDK中，join和future采用的就是该模式
* 因为一个线程需要等待另一个线程的执行结果，所以归结于同步模式

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220303155836268.png)

## 0x10. 生产者消费者模式

* 与前面的保护性暂停中的GuardObjct不同，不需要产生结果和消费结果的线程一一对应
* 消费队列可以用来平衡生产和消费的线程资源
* 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
* 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
* JDK中各种阻塞队列，采用的就是这种模式

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220303162257793.png)

## 0x0A. wait和notify

> https://blog.csdn.net/m0_37989980/article/details/111412907#t0

## 0x0B. park和unpark

> https://blog.csdn.net/m0_37989980/article/details/111412907#t8

## 0x0C. CAS的特点

结合CAS和volatile可以实现无锁并发，适用于线程数少、多核CPU的场景下。

* CAS是基于乐观锁的思想（实际上并不是锁）：最乐观的估计，不怕别的线程来修复共享变量，就算改了也没关系，重试即可

* synchronized是基于悲观锁的思想：最悲观的估计，得防着其他线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会

* CAS体现的是无锁并发，无阻塞并发

  * 因为没有使用synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
  * 但是如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响

## 0x0D. 原子引用ABA问题

> 主线程仅能判断出共享变量的值与初值A是否相同，不能感知到这种从A改为B又改回A的情况，如果主线程希望：
>
> 只要有其他线程【动过了】共享变量，那么自己的cas就算失败，这时仅比较值是不够的，还需要再加一个版本号

1. 通过AtomicStampedReference判断是否更改了版本号，传入的是整型变量
2. 通过AtomicMarkableReference判断是否被修改，传入的是布尔变量

## 0x0E. i++是否线程安全

> 提到这个问题得区分i是成员变量/静态变量还是局部变量，如果是前者需要考虑，对于局部变量不管是基本类型还是包装类型都不需要考虑，包装类型比如Integer是不可变类，是线程安全的。
>
> 假设有1000个线程对i执行++操作，理论上ide结果应该是1000，实际并不是

```java
// i++ 的字节码指令，此时i是一个静态变量
getstatic    i // 获取静态变量i的值
iconst_1	   // 准备常量1
iadd		   // 自增
putstatic    i // 将修改后的值存入静态变量i
```

每个线程都有自己的工作内存，每个线程需要用共享变量时必须先把共享变量从主存load到自己的工作内存，等完成对共享变量的操作时再save到主内存。

问题就出在一个线程读取主存的值后运算完还未刷回主存就被其他线程从主存中读取到了，这时候其他线程读取的数据就是脏数据了。

这也是经典的内存不可见问题，把count加上volatile也不能解决这个问题。因为volatile只能保证可见性并不能保证原子性。多个线程同时读取这个共享变量的值，就算保证其他线程的可见性，也不能保证线程之间读取到同样的值然后互相覆盖对方值的情况。

**解决方案**

1. 对i++操作的方法加同步锁，同时只能由一个线程执行i++
2. 使用支持原子类型操作的类，比如AtomicInteger，内部使用的是CAS
