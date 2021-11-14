---
title: yield方法.md
date: 2020-08-10 22:46:35
categories: 并发
tags: 并发基础
---



### yield方法的使用

* Thread类中的静态native方法
* 让出剩余的时间片，本身进入就绪状态,不释放锁
* cpu调度还可能调度到本线程
* sleep是在一段时间内进入阻塞状态，cpu不会调度它。而yield是让出执行权，本身还处于**就绪状态**，cpu还可能立即调度它

### yield方法的定义

```java
 public static native void yield();
```

### 栗子1

```java
public class YieldDemo02 extends Thread {
    public static Object lock = new Object();

    @Override
    public void run() {
        synchronized (lock) {
            System.out.println(this.getName() + " yield");
            // 打开关闭此注释查看输出效果，对比差异
            this.yield();
		   System.out.println(this.getName() + " run over");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            YieldDemo02 demo = new YieldDemo02();
            demo.start();
        }
    }
}
```

执行结果：

yield是有建议权的，一旦cpu采取了它的建议，就会让出时间片。

但是执行结果却是“间接”的，无论线程的数量有多少，执行结果都是间接有序的，这是为什么呢？

因为有synchronized关键字，synchronized保证原子性。而另一方面，也说明了yield()方法只是让出剩余的时间片，但是**不释放锁**。

```java
Thread-0 yield
Thread-0 run over
Thread-4 yield
Thread-4 run over
```

### await与yield的对比

* wait也是让出执行权，它与yield的区别是，wait会释放锁
* wait需要配合使用synchronized关键字和notifyAll()使用

```java
public class YieldDemo02 extends Thread {
    public static Object lock = new Object();

    @Override
    public void run() {
        synchronized (lock) {
            System.out.println(this.getName() + " yield");

            //使用wait方法来做对比，查看释放锁与不释放锁的区别
            try {
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(this.getName() + " run over");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            YieldDemo02 demo = new YieldDemo02();
            demo.start();
        }

        // 配合wait使用看效果
        synchronized (lock) {
            lock.notifyAll();
        }
    }
}
```

执行结果：

输出结果是先输出yield，再输出run over。且先输出yield的后输出run over。

可以看出，虽然在synchronized代码块内，但是wait方法是释放锁的。

```java
Thread-0 yield
Thread-1 yield
Thread-1 run over
Thread-0 run over
```

总结：

* yield方法最好不要使用，JDK也不推荐使用，但是保留这个方法的目的是在一些包中会有使用到，我们自己尽量不要去使用