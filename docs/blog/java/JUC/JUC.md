# JUC并发编程

## 1 、什么是JUC

JUC就是java.util .concurrent工具包的简称

<img style="display: block; margin: 0 auto;zoom: 75%;" src="blog/java/JUC/picture/image-20210302163205022.png"/>



**普通的线程代码 Thread ：3中创建方式**

```java
package com.juc;

import java.util.concurrent.*;

/**
 * 线程创建的三种方式及区别:
 *  1.继承Thread类
 *  　　 （1）定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。
 * 　　  （2）创建Thread子类的实例，即创建了线程对象。
 * 　　  （3）调用线程对象的start()方法来启动该线程。
 *  2.实现Runnable接口
 *      （1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。
 *      （2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
 *      （3）调用线程对象的start()方法来启动该线程。
 *  3.实现Callable接口
 *      （1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
 *      （2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。
 *      （3）使用FutureTask对象作为Thread对象的target创建并启动新线程。
 *      （4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
 */
public class CreatedThread {
    public static void main(String[] args) throws InterruptedException, TimeoutException, ExecutionException {
        // Thread调用
        TestThread test1 = new TestThread();
        test1.start();
        // Runnable调用
        TestRunnable test2 = new TestRunnable();
        Thread t1 = new Thread(test2);
        t1.start();
        // Callable调用
        TestCallable test3 = new TestCallable();
        FutureTask<Callable> task = new FutureTask<Callable>(test3);
        Thread threadCallable = new Thread(task);
        threadCallable.sleep(10);//等待线程执行结束
        //task.get() 获取call()的返回值。若调用时call()方法未返回，则阻塞线程等待返回值
        //get的传入参数为等待时间，超时抛出超时异常；传入参数为空时，则不设超时，一直等待
        System.out.println(task.get(2, TimeUnit.SECONDS));
    }
}

// 继承Thread类
class TestThread extends Thread {
    @Override
    public void run() {
        System.out.println("通过继承Thread，线程号:" + currentThread().getName());
    }
}

// 实现Runnable接口
class TestRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("通过实现Runnable接口，线程号:" + Thread.currentThread().getName());
    }
}

// 实现Callable接口
class TestCallable implements Callable {
    @Override
    public Object call() throws Exception {
        System.out.println("通过实现Callable接口，线程号:" + Thread.currentThread().getName());
        return "Callable";
    }
}
```

> `补充:`
>
>  **start（）和run（）的区别**
>
> - start()方法用来，开启线程，但是线程开启后并没有立即执行，他需要获取cpu的执行权才可以执行
> - run()方法是由jvm创建完本地操作系统级线程后回调的方法，不可以手动调用（否则就是普通方法）
>
> **Runnable** 没有返回值、效率相比入 Callable 相对较低！



------

## 2 、线程和进程

> `进程`：一个程序，QQ.exe Music.exe 程序的集合；
>
> 一个进程往往可以包含多个线程，至少包含一个！
>
> Java默认有几个线程？ 2 个 mian、GC
>
> `线程`：开了一个进程 Typora，写字，自动保存（线程负责的）
>
> 对于Java而言：Thread、Runnable、Callable

`补充:`Java 无法自己开启线程

<img style="display: block; margin: 0 auto;zoom: 75%;" src="blog/java/JUC/picture/image-20210302164710123.png"/>

#### 并发、并行

`并发`（多线程操作同一个资源）

- CPU 一核 ，模拟出来多条线程，天下武功，唯快不破，快速交替

`并行`（多个人一起行走）

- CPU 多核 ，多个线程可以同时执行； 线程池

> `获取CUP核数`
>
> ```
> public class GainCpuQty {
>     public static void main(String[] args) {
>         // 获取CUP核数
>         System.out.println("CUP核数:" + Runtime.getRuntime().availableProcessors());
>     }
> }
> ```

#### 线程有几个状态

<img style="display: block; margin: 0 auto;zoom: 75%;" src="blog/java/JUC/picture/image-20210302170447092.png"/>

> `新生 NEW`  /  `运行 RUNNABLE`  /  `阻塞 BLOCKED`  /  `等待，死死地等 WAITING`  /  `超时等待 TIMED_WAITING`  /  `终止 TERMINATED`

#### wait/sleep 区别

| 区别               |            wait            |     sleep      |
| ------------------ | :------------------------: | :------------: |
| **来自不同的类**   |           Object           |     Thread     |
| **锁的释放**       |          会释放锁          |    不会释放    |
| **使用的范围不同** | 同步控制方法或者同步控制块 |    任何地方    |
| **需要捕获异常**   |       不需要捕获异常       | 必须要捕获异常 |

------

## 3 、Lock锁（point）

简单卖票应用

```java
package com.juc;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 线程就是一个单独的资源类，没有任何附属的操作！
 */
public class Ticket {
    public static void main(String[] args) {
        // 多线程操作同一个资源类, 把资源类丢入线程
        // TicketBySynchronized bySynchronized = new TicketBySynchronized();
        TicketByLock bySynchronized = new TicketByLock();
        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                bySynchronized.saleTicket();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                bySynchronized.saleTicket();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                bySynchronized.saleTicket();
            }
        }, "C").start();
    }
}

// 使用传统 Synchronized
class TicketBySynchronized {
    private int ticketNum = 50;

    // 使用Synchronized，用锁，类似队列
    public synchronized void saleTicket(){
        if(ticketNum > 0){
            System.out.println(Thread.currentThread().getName() + "卖出了"+(ticketNum--)+"票,剩余："+ticketNum);
        }
    }
}

// 使用 Lock
class TicketByLock {
    private int ticketNum = 50;
    Lock lock = new ReentrantLock();

    public void saleTicket(){
        lock.lock(); // 加锁

        try {
            if(ticketNum > 0){
                System.out.println(Thread.currentThread().getName() + "卖出了"+(ticketNum--)+"票,剩余："+ticketNum);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); // 释放
        }
    }
}
```



#### Synchronized 和 Lock 区别

|                  Synchronized                   |                     Lock                      |
| :---------------------------------------------: | :-------------------------------------------: |
|                内置的Java关键字                 |                 是一个Java类                  |
|              无法判断获取锁的状态               |            可以判断是否获取到了锁             |
|                  会自动释放锁                   |     必须要手动释放锁！如果不释放锁，死锁      |
| 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等） |             锁就不一定会等待下去              |
|         可重入锁，不可以中断的，非公平          | 可重入锁，可以 判断锁，非公平（可以自己设置） |
|            适合锁少量的代码同步问题             |             适合锁大量的同步代码              |

