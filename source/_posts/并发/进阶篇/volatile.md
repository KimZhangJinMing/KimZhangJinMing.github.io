---
title: volatile.md
date: 2020-08-10 22:46:35
categories: 并发
tags: JUC
---



### volatile保证可见性，不保证原子性

* volatile强制线程到共享内存中读取数据，而不从线程工作内存中读取，从而使变量在多个线程中可见
* volatile无法保证原子性，volatile属于轻量级的同步，性能比synchronized强很多(不加锁)，但是只 保证线程见的可见性，并不能替代synchronized的同步功能，netty框架中大量使用了volatile 

#### 栗子1：volatile不能保证原子性

sum属性加了volatile关键字

```java
public class ThreadDemo14 implements Runnable {

    public volatile static Integer sum = 0;

    public static void add() {
        System.out.println(Thread.currentThread().getName() + "初始sum=" + sum);
        for (int i = 0; i < 10000; i++) {
            sum++;
        }
        System.out.println(Thread.currentThread().getName() + "计算后sum=" + sum);
    }

    @Override
    public void run() {
        add();
    }

    public static void main(String[] args) {
        //如果volatile具有原子性,那么10个线程并发调用，最终结果应该为100000
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            es.submit(new ThreadDemo14());
        }
        es.shutdown();
        while (true) {
            if (es.isTerminated()) {
                System.out.println("sum最终=" + sum);
                if (sum == 100000) {
                    System.out.println(sum + "=ok");
                } else {
                    System.out.println(sum + "=no");
                }
                break;
            }
        }
    }
}
```

运行结果:

多次运行可能结果都不一样。因为volatile不能保证原子性，可能线程A把sum改变了，还没来得及设置回主内存，又被其他线程给读取了。

```java
pool-1-thread-1初始sum=0
pool-1-thread-3初始sum=0
pool-1-thread-2初始sum=0
pool-1-thread-5初始sum=0
pool-1-thread-3计算后sum=4486
pool-1-thread-1计算后sum=6411
pool-1-thread-4初始sum=7627
pool-1-thread-2计算后sum=9046
pool-1-thread-6初始sum=14162
pool-1-thread-7初始sum=17079
pool-1-thread-4计算后sum=19511
pool-1-thread-8初始sum=21795
pool-1-thread-5计算后sum=24844
pool-1-thread-6计算后sum=26518
pool-1-thread-9初始sum=28104
pool-1-thread-10初始sum=29595
pool-1-thread-8计算后sum=33121
pool-1-thread-7计算后sum=33111
pool-1-thread-9计算后sum=41465
pool-1-thread-10计算后sum=42512
sum最终=42512
42512=no
```

### volatile与static区别

* static保证唯一性，不保证一致性，多个实例共享一个静态变量
* volatile保证一致性，不保证唯一性，多个实例有多个volatile变量

### Atomic类的原子性

==**Actomic类采用了CAS这种非锁机制**==

 #### 栗子1：使用AtomicInteger等原子类可以保证共享变量的原子性 

```java
public class ThreadDemo14 implements Runnable {

    public static AtomicInteger sum = new AtomicInteger(0);

    public static void add() {
        System.out.println(Thread.currentThread().getName() + "初始sum=" + sum);
        for (int i = 0; i < 10000; i++) {
            sum.addAndGet(1);
        }
        System.out.println(Thread.currentThread().getName() + "计算后sum=" + sum);
    }

    @Override
    public void run() {
        add();
    }

    public static void main(String[] args) {
        //如果volatile具有原子性,那么10个线程并发调用，最终结果应该为100000
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            es.submit(new ThreadDemo14());
        }
        es.shutdown();
        while (true) {
            if (es.isTerminated()) {
                System.out.println("sum最终=" + sum);
                if (sum.get() == 100000) {
                    System.out.println(sum + "=ok");
                } else {
                    System.out.println(sum + "=no");
                }
                break;
            }
        }
    }
}
```

执行结果：

不管运行多少次，结果都是正确的

```java
pool-1-thread-1初始sum=0
pool-1-thread-3初始sum=0
pool-1-thread-2初始sum=0
pool-1-thread-5初始sum=14727
pool-1-thread-10初始sum=19921
pool-1-thread-3计算后sum=26493
pool-1-thread-6初始sum=32980
pool-1-thread-1计算后sum=35974
pool-1-thread-2计算后sum=41300
pool-1-thread-4初始sum=54770
pool-1-thread-10计算后sum=56532
pool-1-thread-6计算后sum=59183
pool-1-thread-7初始sum=60117
pool-1-thread-8初始sum=61625
pool-1-thread-5计算后sum=68908
pool-1-thread-9初始sum=79628
pool-1-thread-4计算后sum=82182
pool-1-thread-7计算后sum=92269
pool-1-thread-8计算后sum=95401
pool-1-thread-9计算后sum=100000
sum最终=100000
100000=ok
```

#### 栗子2： Atomic类不能保证成员方法的原子性 

```java
public class ThreadDemo15 implements Runnable {
    //原子类
    private static AtomicInteger sum = new AtomicInteger(0);

    // 如果add方法是原子性的,那么每次的结果都是10的整数倍
    // synchronized加锁可以保证原子性
    public static void add() {
        sum.addAndGet(1);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        sum.addAndGet(9);
        System.out.println(sum);
    }

    @Override
    public void run() {
        add();
    }

    public static void main(String[] args) {
        //10个线程调用，每个线程得到10的倍数， 最终结果应该为100，才是正确的
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            es.submit(new ThreadDemo15());
        }
        es.shutdown();
    }

}
```

执行结果：

```java
28
100
91
82
37
73
64
55
46
28
```

使用synchronized可以保证方法的原子性。

```java
public synchronized static void add() {
    sum.addAndGet(1);
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    sum.addAndGet(9);
    System.out.println(sum);
}
```

执行结果：

```java
10
20
30
40
50
60
70
80
90
100
```

### CAS（Compare and swap）

*  JDK提供的非阻塞原子操作，通过硬件保证了比较、更新操作的原子性 
* JDK的Unsafe类提供了一系列的compareAndSwap*方法来支持CAS操作

#### 解决ABA问题：

* 给变量分配时间戳、版本来解决ABA问题
* JDK中使用java.util.concurrent.atomic.AtomicStampedReference类给每个变量的状态都分配一个时间 戳，避免ABA问题产生。

#### 栗子1：ABA问题

![image-20200626214300170](D:\ming\images\image-20200626214300170.png)

```java
public class CasDemo0 {
    private static AtomicStampedReference<Integer> atomic = new AtomicStampedReference<>(100, 0);

    public static void main(String[] args) throws InterruptedException {
        Thread t0 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
                boolean sucess = atomic.compareAndSet(100, 101, atomic.getStamp(), atomic.getStamp() + 1);
                System.out.println(Thread.currentThread().getName() + " set 100>101 : " + sucess);
                sucess = atomic.compareAndSet(101, 100, atomic.getStamp(), atomic.getStamp() + 1);
                System.out.println(Thread.currentThread().getName() + " set 101>100 : " + sucess);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        t0.start();

        Thread t1 = new Thread(() -> {
            try {
                int stamp = atomic.getStamp();
                System.out.println(Thread.currentThread().getName() + " 修改之前 : " + stamp);
                TimeUnit.SECONDS.sleep(2);
                int stamp1 = atomic.getStamp();
                System.out.println(Thread.currentThread().getName() + " 等待两秒之后,版本被t0线程修改为 : " + stamp1);

                // 以下两次修改都不会成功,因为版本不符,虽然期待值是相同的,因此解决了ABA问题
                boolean success = atomic.compareAndSet(100, 101, stamp, stamp + 1);
                System.out.println(Thread.currentThread().getName() + " set 100>101 使用错误的时间戳: " + success);
                success = atomic.compareAndSet(101, 100, stamp, stamp + 1);
                System.out.println(Thread.currentThread().getName() + " set 101>100 使用错误的时间戳: " + success);

                // 以下修改是成功的,因为使用了正确的版本号,正确的期待值
                success = atomic.compareAndSet(100, 101, stamp1, stamp1 + 1);
                System.out.println(Thread.currentThread().getName() + " set 100>101 使用正确的时间戳: " + success);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        t1.start();

        t0.join();
        t1.join();

        System.out.println("main is over");
    }
}
```

执行结果：

```java
Thread-1 修改之前 : 0
Thread-0 set 100>101 : true
Thread-0 set 101>100 : true
Thread-1 等待两秒之后,版本被t0线程修改为 : 2
Thread-1 set 100>101 使用错误的时间戳: false
Thread-1 set 101>100 使用错误的时间戳: false
Thread-1 set 100>101 使用正确的时间戳: true
main is over
```

