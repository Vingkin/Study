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
>
> 下图RUNNABLE中的阻塞状态应该去除

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/image-20220225202815248.png)

**线程的状态转换**

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

## 0x0D. wait()和notify()

> Object类中的方法
>
> https://blog.csdn.net/m0_37989980/article/details/111412907#t0

## 0x0E. sleep()和wait()的区别

1. sleep是Thread方法，wait是Object方法
2. sleep不需要强制和synchronized配合使用，但wait需要和synchronized一起使用
3. **sleep不会释放锁对象，wait会释放锁对象**

他们的线程状态都是`TIMED_WAITING`

## 0x0F. 保护性暂停模式

> 用于一个线程等待另一个线程的执行结果
>
> join()内部采用的就是这个原理，不过join()中是一个线程等待另一个线程结束

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

## 0x11. park()和unpark()

> https://blog.csdn.net/m0_37989980/article/details/111412907#t8

* park和unpark是LockSupport类中的方法，运行时会调用Unsafe类中的native方法
* 每个线程都会和一个park对象关联起来，由三部分组成`_counter`,`_cond`,`_mutex_`。核心部分是counter，可以理解为一个标记位。
* 当调用park时会查看counter是否为0，为0则进入cond阻塞。为1则继续运行并将counter置为0。
* 当调用unpark时，会将counter置为1，若之前的counter值为0，还会唤醒阻塞的线程。
* **如果先调用unpark再调用park不会阻塞线程。调用unpark后将counter置为1，再调用park线程发现counter为1继续运行并将counter置为0。**

**park()&unpark()与wait()&notify()对比**

1. wait，notify和notifyAll必须配合Object Monitor(synchronized)一起使用，而park和unpark不必
2. park，unpark是以线程为单位来【阻塞】和【唤醒】线程，而notify只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，无法唤醒指定的线程。
3. ~~park，unpark可以先unpark，而wait，notify不能先notify~~

## 0x12. 死锁，活锁，饥饿

### 死锁

代码演示：

```java
public static void main(String[] args) {
	final Object A = new Object();
	final Object B = new Object();
	
	new Thread(()->{
		synchronized (A) {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			synchronized (B) {

			}
		}
	}).start();

	new Thread(()->{
		synchronized (B) {
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			synchronized (A) {

			}
		}
	}).start();
}
```

**发生死锁的必要条件：**

1. 互斥条件：在一段时间内，一种资源只能被一个线程所使用
2. 请求和保持条件：线程已经拥有了至少一种资源，同时又去申请其他资源。因为其他资源被别的线程所使用。该线程进入阻塞状态同时不释放自己已有的资源。
3. 不可抢占条件：进程对已获得的资源在未使用完成前不能被抢占，之恶能在线程使用完后自己释放。
4. 循环等待条件：发生死锁时，必然存在一个线程---资源的循环链

**定位死锁的方法：**

1. `jstack + 进程id`命令查看线程状态有Java层面死锁线程信息
2. `jconsole`有死锁检测功能

**避免死锁的方法：**

- 在线程使用锁对象时, 采用**固定加锁的顺序**, 可以使用Hash值的大小来确定加锁的先后
- 尽可能缩减加锁的范围, 等到操作共享变量的时候才加锁
- 使用可释放的定时锁 (一段时间申请不到锁的权限了, 直接释放掉)

### 活锁

- `活锁`出现在两个线程 **`互相改变对方的结束条件`**，谁也无法结束。

**避免活锁的方法：**

* 在线程执行时，中途给予不同的间隔时间, 让某个线程先结束即可。

**死锁与活锁的区别：**

* 死锁是因为线程互相持有对象想要的锁，并且都不释放，最后到时线程阻塞，停止运行的现象。
* 活锁是因为线程间修改了对方的结束条件，而导致代码一直在运行，却一直运行不完的现象。

### 饥饿

- 某些线程因为优先级太低，导致一直无法获得资源的现象。
- 在使用`顺序加锁`时，可能会出现`饥饿现象`

## 0x13. 固定线程运行顺序

**wait()&notify()**

```java
public class Test {

    static final Object lock = new Object();
    static boolean t2runned = false;

    public static void main(String[] args) {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    while (!t2runned) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    log.debug("1");
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized(lock) {
                    log.debug("2");
                    t2runned = true;
                    lock.notify();
                }
            }
        }, "t2");

        t1.start();
        t2.start();

    }
}
```

**park()&unpark()**

```java
public class Test {

    public static void main(String[] args) {
        
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                LockSupport.park();
                log.debug("1");
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                log.debug("2");
                LockSupport.unpark(t1);
            }
        }, "t2");

        t1.start();
        t2.start();

    }
}
```

**await()&signal()**

```java
public class Test {

    private static ReentrantLock lock = new ReentrantLock();
    private static boolean t2runned = false;
    static Condition condition1 = lock.newCondition();

    public static void main(String[] args) {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                try {
                    while (!t2runned) {
                        try {
                            condition1.await();
                            log.debug("1");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                } finally {
                    lock.unlock();
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                try {
                    log.debug("2");
                    t2runned = true;
                    condition1.signal();
                } finally {
                    lock.unlock();
                }
            }
        }, "t2");

        t1.start();
        t2.start();

    }
}
```

## 0x14. 线程交替输出

**wait()&notify()**

```java
public class Test {

    static boolean t1runned = false;
    static boolean t2runned = true;
    static final Object lock = new Object();

    public static void main(String[] args) {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    synchronized (lock) {
                        while (!t2runned) {
                            try {
                                lock.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        log.debug("1");
                        t1runned = true;
                        t2runned = false;
                        lock.notify();
                    }
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    synchronized (lock) {
                        while (!t1runned) {
                            try {
                                lock.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        log.debug("2");
                        t1runned = false;
                        t2runned = true;
                        lock.notify();
                    }
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

**park()&unpark()**

```java
public class Test {

    volatile static boolean t1runned = false;
    volatile static boolean t2runned = true;
    static final Object lock = new Object();
    static Thread t1;
    static Thread t2;

    public static void main(String[] args) {

        t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    if (!t2runned) {
                        LockSupport.park();
                    }
                    log.debug("1");
                    t2runned = false;
                    t1runned = true;
                    LockSupport.unpark(t2);
                }
            }
        }, "t1");

        t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    if  (!t1runned) {
                        LockSupport.park();
                    }
                    log.debug("2");
                    t2runned = true;
                    t1runned = false;
                    LockSupport.unpark(t1);
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

**await()&signal()**

```java
public class Test {

    static boolean t1runned = false;
    static boolean t2runned = true;

    public static void main(String[] args) {

        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    reentrantLock.lock();
                    try {
                        while (!t2runned) {
                            condition.await();
                        }
                        log.debug("1");
                        t1runned = true;
                        t2runned = false;
                        condition.signal();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        reentrantLock.unlock();
                    }
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    reentrantLock.lock();
                    try {
                        while (!t1runned) {
                            condition.await();
                        }
                        log.debug("2");
                        t1runned = false;
                        t2runned = true;
                        condition.signal();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        reentrantLock.unlock();
                    }
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

## 0x15. 并发编程的三大特性

* 原子性：保证指令不会受到线程上下文切换的影响。**程序的原子性是指整个程序中的所有操作，要么全部完成，要么全部失败，不可能滞留在中间某个环节；在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程所打断。**
* 可见性：保证指令不会受cpu缓存的影响。**一个线程对共享变量值的修改，能够及时地被其他线程看到**
* 有序性，保证指令不会受到cpu指令并行优化的影响

## 0x16. volatile原理

volatile的底层实现原理是内存屏障

1. 保证可见性
   * 对volatile变量的写指令后会加入写屏障。写屏障保证该屏障之前的，对共享变量的改动都会同步到主存中。
   * 对volatile变量之间会加入读屏障。读屏障保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据。
2. 保证有序性
   * 写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后
   * 读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前

## 0x17. volatile和synchronized

* 一个线程对volatile变量的修改对另一个线程可见，不能保证原子性，仅用在一个写线程，多个读线程的情况。（比如volatile修饰的i，两个线程一个i++一个i--，只能保证看到最新值，不能解决指令交错的问题。）
* synchronized语句块既能保证代码块的原子性，也同时能保证代码块内变量的可见性。但缺点是synchronized属于重量级锁，性能相对较低。
* volatile关键字只能修饰变量，synchronized还可以修饰方法，类以及代码块。
* volatile关键字主要用于解决变量在多个线程之间的可见性，而synchronized关键字解决的是多个线程之间访问资源的同步性。

## 0x18. volatile和synchronized在有序性上的不同

* synchronized的有序性是持有相同锁的两个同步块只能串行的进入，即被加锁的内容要按照顺序被多个线程执行，但是其内部的同步代码还是会发生重排序。
* volatile的有序性是通过插入内存屏障来保证指令按照顺序执行。不会存在后面的指令跑到前面的指令之前来执行。是保证编译器优化的时候不会让指令乱序。
* **synchronized是不能保证指令重排的。**

## 0x19. i++是否线程安全

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

## 0x11A. CAS的特点

结合CAS和volatile可以实现无锁并发，适用于线程数少、多核CPU的场景下。

* CAS是基于乐观锁的思想~~（实际上并不是锁）~~：最乐观的估计，不怕别的线程来修复共享变量，就算改了也没关系，重试即可

* synchronized是基于悲观锁的思想：最悲观的估计，得防着其他线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会

* CAS体现的是无锁并发，无阻塞并发

  * 因为没有使用synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
  * 但是如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响

## 0x1B. Atomic原子类

> [并发编程面试必备：JUC 中的 Atomic 原子类总结 (qq.com)](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484834&idx=1&sn=7d3835091af8125c13fc6db765f4c5bd&source=41#wechat_redirect)

1. 原子整数

   1. AtomicInteger
   2. AtomicLong
   3. AtomicBoolean

   ```java
   public static void main(String[] args) {
       AtomicInteger i = new AtomicInteger(0);
       
       // 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
       System.out.println(i.getAndIncrement());
       
       // 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
       System.out.println(i.incrementAndGet());
       
       // 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
       System.out.println(i.decrementAndGet());
       
       // 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
       System.out.println(i.getAndDecrement());
       
       // 获取并加值（i = 0, 结果 i = 5, 返回 0）
       System.out.println(i.getAndAdd(5));
       
       // 加值并获取（i = 5, 结果 i = 0, 返回 0）
       System.out.println(i.addAndGet(-5));
       
       // 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
       // 函数式编程接口，其中函数中的操作能保证原子，但函数需要无副作用
       System.out.println(i.getAndUpdate(p -> p - 2));
       
       // 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
       // 函数式编程接口，其中函数中的操作能保证原子，但函数需要无副作用
       System.out.println(i.updateAndGet(p -> p + 2));
       
       // 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
       // 函数式编程接口，其中函数中的操作能保证原子，但函数需要无副作用
       // getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
       // getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
       System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
       
       // 计算并获取（i = 10, p 为 i 的当前值, x 为参数1值, 结果 i = 0, 返回 0）
       // 函数式编程接口，其中函数中的操作能保证原子，但函数需要无副作用
       System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
   }
   ```

2. 原子引用

   > 原子引用的作用: **保证引用类型的共享变量是线程安全的(确保这个原子引用没有引用过别人)**

   1. AtomicReference
   2. AtomicStampedReference
   3. AtomicMarkableReference

3. 原子数组

   > 保证数组内元素的线程安全

   1. AtomicIntegerArray
   2. AtomicLongArray
   3. AtomicReferenceArray

4. 字段更新器

   > 保证`多线程`访问`同一个对象的成员变量`时, `成员变量的线程安全性`。

   1. AtomicIntegerFieldUpdater
   2. AtomicLongFieldUpdater
   3. AtomicReferenceFieldUpdater

5. 原子累加器

   1. LongAdder
   2. LongAccumulator
   3. DoubleAdder
   4. DoubleAccumulator

## 0x1C. 原子引用ABA问题

> 采用CAS主线程仅能判断出共享变量的值与初值A是否相同，不能感知到这种从A改为B又改回A的情况，如果主线程希望：
>
> 只要有其他线程【动过了】共享变量，那么自己的cas就算失败，这时仅比较值是不够的，还需要再加一个版本号

1. 通过AtomicStampedReference判断是否更改了版本号，传入的是整型变量
2. 通过AtomicMarkableReference判断是否被修改，传入的是布尔变量

## 0x1D. LongAdder原理

```java
// 累加单元数组，懒惰初始化
transient volatile Cell[] cells;
// 基础值，如果没有竞争，则用cas累加这个域
transient volatile long base;
// 在cells创建或扩容时，置为1，表示加锁
transient volatile int cellsBusy;
```

**性能提升的原因很简单，就是在有竞争时，设置多个`累加单元`(但不会超过cpu的核心数)，Therad-0 累加 Cell[0]，而 Thread-1 累加Cell[1]… 最后将结果汇总。这样它们在累加时操作的不同的 Cell 变量，`因此减少了 CAS 重试失败`，从而提高性能。**

**之前AtomicLong等都是在一个`共享资源变量`上进行竞争, `while(true)`循环进行CAS重试, 性能没有`LongAdder`高**

## 0x1E. Unsafe

> Unsafe并不是表示线程不安全，而是表示Unsafe类中的操作不安全，因为是对于底层的操作。
>
> Unsafe对象提供了非常底层的，操作系内存、线程的方法，Unsafe对象不能直接调用，只能通过反射获得

```java
Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
theUnsafe.setAccessible(true);
Unsafe unsafe = (Unsafe) theUnsafe.get(null);
System.out.println(unsafe);
```

## 0x1F. 不可变类

**final的使用**

* 属性用final修饰保证该属性是只读的，不能修改
* 类用final修饰保证了类不能被继承，该类中的方法不能被重写，防止子类无意间破坏不变性

**保护性拷贝**

使用字符串时，也有一些跟修改相关的方法啊，比如`substring、replace` 等，那么下面就看一看这些方法是 如何实现的，就以 substring 为例：

```java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    // 上面是一些校验，下面才是真正的创建新的String对象
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

发现其方法最后是调用String 的构造方法创建了一个新字符串，再进入这个构造看看，是否对 `final char[] value` 做出了修改：结果发现也没有，构造新字符串对象时，会生成新的 `char[] value`，对内容进行复制。
这种通过创建副本对象来避免共享的手段称之为【保护性拷贝（defensive copy）】

## 0x20. final原理

```java
public class TestFinal {
	final int a = 20; 
}
```

```java
0: aload_0
1: invokespecial #1 // Method java/lang/Object."<init>":()V
4: aload_0
5: bipush 20
7: putfield #2 // Field a:I
 <-- 写屏障
10: retu
```

发现 final 变量的赋值也会通过 **putfield** 指令来完成，同样在这条指令之后也会加入`写屏障`，**保证在其它线程读到它的值时不会出现为 0 的情况。**

* 写屏障保证该屏障之前的，对共享变量的改动都会同步到主存中。
* 写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后

## 0x21. 享元模式

享元模式简单理解就是重用数量有限的同一对象，比如字符串常量池，包装类常量池，线程池以及字符串连接池都运用了享元模式的思想。

## 0x22. 线程池

### 线程池的好处

1. 降低资源消耗。通过重复利用已创建的线程来降低线程创建和销毁所带来的消耗。
2. 提高响应速度。当任务到达时，如果有空闲线程，任务可以不需要等到线程创建就直接运行。
   1. 提高线程的客观理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 线程池状态

- ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量。使用一个AtomicInteger来表示状态和数量，**可以通过一次CAS同时更改两个属性的值**。

|  状态名称  | 高3位的值 |                        描述                         |
| :--------: | :-------: | :-------------------------------------------------: |
|  RUNNING   |    111    |        接收新任务，同时处理任务队列中的任务         |
|  SHUTDOWN  |    000    |       不接受新任务，但是处理任务队列中的任务        |
|    STOP    |    001    |    中断正在执行的任务，同时抛弃阻塞队列中的任务     |
|  TIDYING   |    010    | 任务执行完毕，活动线程为0时，即将进入TERMINATED状态 |
| TERMINATED |    011    |                      终结状态                       |

### ThreadPoolExecutor参数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)

```

* corePoolSize：核心线程数
* maximumPoolSize：最大线程数
  * maximumPoolSize - corePoolSize = 救急线程数
  * **救急线程在没有空闲核心线程和任务队列满了的情况下才会创建使用**
* keepAliveTime：救急线程空闲时的最大空闲时间
* unit：时间单位，针对救急线程
* workQueue：阻塞队列
  * 有界阻塞队列：ArrayBlockingQueue
  * 无界阻塞队列：LinkedBlockingQueue
  * 最多只有一个任务的阻塞队列：SynchronizedQueue
  * 优先队列：PriorityBlockingQueue
* ThreadFactory：线程工厂（给线程取名字）
* handler：拒绝策略（当活动线程数==最大线程数且阻塞队列满的情况下采取的策略）

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/20210202214622633.png)



### 拒绝策略

> 当活动线程数等于最大线程数且阻塞队列满的情况下采取的策略

JDK提供了四种实现

1. **AbortPolicy终止策略**：丢弃该任务并抛出RejectedExecutionException异常。**这是默认策略**
2. **DiscardPolicy丢弃策略**：丢弃任务，但是不抛出异常。如果任务队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。
3. **DiscardOldestPolicy弃老策略**：丢弃队列最前面的任务，然后重新提交被拒绝的任务
4. **CallerRunsPolicy调用者运行策略**：由调用者线程自行处理该任务

### Executors创建的线程池

> 由`Executors类`提供的工厂方法来创建线程池！`Executors` 是Executor 框架的工具类
>
> 一般不适用，而是直接使用ThreadPoolExecutor构造方法



`newFixedThreadPool`

```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

**特点**

1. 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间
2. `阻塞队列是无界的，可以放任意数量的任务`
3. **适用于任务量已知，相对耗时的任务**



`newCachedThreadPool`

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

特点

* 没有核心线程，最大线程数为Integer.MAX_VALUE，所有创建的线程都是救急线程 (可以无限创建)，空闲时生存时间为60秒
* 阻塞队列使用的是SynchronousQueue
  * SynchronousQueue是一种特殊的队列
    * 没有容量，没有线程来取是放不进去的
    * 只有当线程取任务时，才会将任务放入该阻塞队列中
* 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线程。 适合任务数比较密集，但每个任务执行时间较短的情况
* 

`newSingleThreadExecutor`

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

使用场景：

1. 希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。 任务执行完毕，这唯一的线程也不会被释放。
2. 区别：
   1. 和自己创建单线程执行任务的区别：自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而`newSingleThreadExecutor`线程池还会新建一个线程，保证池的正常工作
   2. `Executors.newSingleThreadExecutor()` 线程个数始终为1，不能修改
      1. FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 `ExecutorService `接口，因此不能调用 `ThreadPoolExecutor `中特有的方法
3. 和`Executors.newFixedThreadPool(1)` 初始时为1时的区别：`Executors.newFixedThreadPool(1)` 初始时为1，以后还可以修改，对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改

### 执行 execute()方法和 submit()方法的区别是什么呢？

> 就像runnable()和callable()的区别，submit()有返回值返回一个Future的对象。

### 线程池创建多少线程合适

> 下面两点只是纯理论说法，具体个数要是需要测试得到

1. CPU密集型

   通常采用 **`cpu 核数 + 1`** 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费

2. IO密集型

   CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。通过CPU的利用率计算得到。


