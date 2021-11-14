---
title: JOIN方法.md
date: 2020-08-10 22:46:35
categories: 并发
tags: 并发基础
---


### JOIN方法的使用

> 应用场景

阻塞当前线程，等待调用JOIN方法的线程执行结束后才继续执行。

因为新的线程加入了我们，所以我们要等他执行完再出发。

> 混淆点

* JOIN方法是Thread类的方法，而notify()、notifyall()、wait()等方法是Object类的方法

* 主线程等待其他线程执行完的时候，主线程的状态是WAITING

> 方法的定义

不带参数的join方法表示必须等调用JOIN方法的线程执行结束后，才继续执行当前线程。

方法内部调用的是以毫秒为单位的JOIN方法。

```java
 public final void join() throws InterruptedException {
        join(0);
    }
```

以毫秒为单位的JOIN方法。

内部实现调用了Object的wait方法。

参数表示如果等待millis毫秒后，你还没执行完成，我就继续往下执行了。

join可以让本线程等待别的线程执行完毕， Thread.join()调用相当于Thread.join(0) ，代码内的逻辑是，先判断如果输入的时间小于0就抛异常，然后就分为两种情况：

第一种是传入0的情况，这也是默认的情况，那么就代表永久等待，直到线程执行完毕，但是我们去看代码，看到了：

while (isAlive()) {

​      wait(0);

​    }

但是哪里有唤醒的行为呢？我们前面讲过，wait(0)代表一直休眠，直到被notify()，那么为什么我们看不到notify()的相关代码，却依然能在线程运行结束后，自动被唤醒呢？

秘密就在Thread类在run方法运行结束后，自动执行notifyAll()，让我们看看一看JVM层代码：

通过对Jvm natvie的源码分析,我们发现thread执行完成后，cpp的源码中会在thread执行完毕后,会调用exit方法,该方法中原来隐含有调用notify_all(thread)的动作:

   void JavaThread::exit(booldestroy_vm,ExitTypeexit_type)；//做清理、收尾工作，

  上面的方法中会调用 ensure_join(this);

  下面是ensure_join方法的源码:

  static void ensure_join(JavaThread*thread){

​     Handle threadObj(thread,thread->threadObj());

​     ObjectLocker lock(threadObj,thread);thread->clear_pending_exception();

​     java_lang_Thread::set_thread_status(threadObj(),java_lang_Thread::TERMINATED);

​     java_lang_Thread::set_thread(threadObj(),NULL);

​     lock.notify_all(thread); //这里执行了notify_all,进行了wait的唤醒

​     thread->clear_pending_exception();

  }

  通过以上分析后，我们就可以有了join的替代写法:

​    synchronized(thA) { 

​       thA.wait(); 

​    }

```java
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
		// 等待的时间为0，只有线程还是运行状态，则永远等待
    	// 这里使用了wait方法，那么在哪里notify/notify呢？
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```

两个参数的JOIN方法，实现对纳秒级别的控制，很少使用。

当纳秒数大于500000，或者纳秒不为0，毫秒为0时，使毫秒数+1

```java
public final synchronized void join(long millis, int nanos)
throws InterruptedException {

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }

    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++;
    }

    join(millis);
}
```

> 注意点

1.如果同时调用多个join，join会有一定的顺序。

2.join是一个native方法

> join方法中使用了wait方法，那么在哪里notify/notify呢？

在每个线程的run方法结束后，会执行notify/notifyall方法

#### 栗子1

创建两个线程，每个线程休眠2s后，输出当前线程状态。

在main线程中调用join方法，等待两个线程执行完毕后继续执行main线程。

```java
public class JoinDemo01 {
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "run over! " + Thread.currentThread().getState());

        });

        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "run over! " + Thread.currentThread().getState());

        });


        thread1.start();
        thread2.start();
        System.out.println(Thread.currentThread() + "are waiting  " + thread1.getName() + " and " + thread2.getName());

        // 这里参数设置为1000，有特别意义
        try {
            System.out.println("join:" + new Date());
            thread1.join(1000);
            System.out.println("join:" + new Date());
            thread2.join(1000);
            System.out.println("join:" + new Date());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("final: " + thread1.getName() + thread1.getState());
        System.out.println("final: " + thread2.getName() + thread2.getState());
    }
}

```

执行结果：

34s的时候调用thread1.join方法，在thread1线程的run方法是sleep2s，参数设置为等待1s后继续往下执行。

即当35s的时候调用thread2.join方法，在thread2线程的run方法是sleep2s，但是在34s的时候thread1,thread2都start了，

所以在36s的时候，thread1，thread2都已经执行完成了。

那么，就会有一个问题，main方法接下来的输出语句和thread1,thread2中run方法的输出语句，哪个先执行？

**这是不确定的，多次运行会发现结果不同，而结果的不同会体现在线程的状态不同**

```java
// 线程先执行完后输出语句
Thread[main,5,main]are waiting  Thread-0 and Thread-1
join:Wed Jun 24 23:59:34 CST 2020
join:Wed Jun 24 23:59:35 CST 2020
Thread-1run over! RUNNABLE
Thread-0run over! RUNNABLE
join:Wed Jun 24 23:59:36 CST 2020
final: Thread-0TERMINATED
final: Thread-1TERMINATED
```

```java
// 先输出了语句，线程再执行完。好像会受输出时间的打印语句影响？？？
Thread[main,5,main]are waiting  Thread-0 and Thread-1
final: Thread-0RUNNABLE
final: Thread-1BLOCKED  // 这里的BLOCKED是IO的阻塞，因为线程的BLOCKED状态只有在ysnchronized的时候才会出现
Thread-0run over! RUNNABLE
Thread-1run over! RUNNABLE
```

***

### 栗子2

```java
public class JoinDemo02 {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " run over!");
        });

        Thread mainThread = Thread.currentThread();

        Thread t2 = new Thread(() -> {
            //调用主线程的interrupt方法, 开启中断标记, 会影响主线中的join方法抛出异常,但是并不会阻碍t1线程的运行
            mainThread.interrupt();
            System.out.println(mainThread.getName() + " interrupt!");
            System.out.println(Thread.currentThread().getName() + " run over!");
        });

        t1.start();
        t2.start();

        System.out.println(Thread.currentThread().getName() + " wait " + t1.getName() + " and " + t2.getName() + " run over!");

        try {
            t1.join();  // 等待t1线程执行完毕，才继续往下执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("final:" + t1.getName() + " and " + t2.getName() + " run over!");

        System.out.println("t1's state:" + t1.getState()); // TIME_WAITING
        System.out.println("t2's state:" + t2.getState()); // TERMINTED
        System.out.println("main's state:" + mainThread.getState()); // RUNNABLE，

    }

}

```

执行结果：

main方法运行后，thread2先运行，thread2中中断了main线程，影响了main线程的join方法，抛出异常，但不会影响thread1的执行。

从异常信息可以看到，中断main线程影响的是main线程中的join方法，thread1仍然正常执行，从thread1的线程状态是TIMED_WAITING可以看出。

```java
main wait Thread-0 and Thread-1 run over!
main interrupt!
Thread-1 run over!
java.lang.InterruptedException
	at java.lang.Object.wait(Native Method)
	at java.lang.Thread.join(Thread.java:1252)
	at java.lang.Thread.join(Thread.java:1326)
	at thread03.JoinDemo02.main(JoinDemo02.java:28)
final:Thread-0 and Thread-1 run over!
t1's state:TIMED_WAITING
t2's state:TERMINATED
main's state:RUNNABLE
Thread-0 run over!
```

接着，我们修改一下线程2的方法：

```java
Thread t2 = new Thread(() -> {
    //修改为t1.interrupt();观察效果, 会影响t1线程的sleep方法抛出异常,让t1线程结束
    t1.interrupt();
    System.out.println(t1.getName() + " interrupt!");
    System.out.println(Thread.currentThread().getName() + " run over!");
});
```

执行结果：

在thread2的方法中中断thread1的执行，这时中断的是thread1的sleep方法，thread1会抛出异常，继续往下执行，不再sleep了。

从异常信息可以看出是thread1的sleep方法被打断了，而且thread1的线程状态是TERMINATED。而main线程的状态是RUNNABLE，对主线程无影响。

```java
main wait Thread-0 and Thread-1 run over!
Thread-0 interrupt!
Thread-1 run over!
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at thread03.JoinDemo02.lambda$main$0(JoinDemo02.java:18)
	at java.lang.Thread.run(Thread.java:748)
Thread-0 run over!
final:Thread-0 and Thread-1 run over!
t1's state:TERMINATED
t2's state:TERMINATED
main's state:RUNNABLE
```

总结：

* interrupt方法可以中断sleep()、join()等方法
* 某个线程如果被interrupt()打断，会抛出InterruptedException，并继续往下执行。可以在抛出异常后，再次调用子线程的interrupt方法，将中断信号传递给子线程。子线程处理中断信号后，就不会继续往下执行了

