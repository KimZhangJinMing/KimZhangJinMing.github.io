---
title: synchronized.md
date: 2020-08-10 22:46:35
categories: 并发
tags: 并发基础
---


### 什么是线程安全

* 当多个线程访问某一个类、对象或方法时，这个类、对象或方法都能表现出与单线程执行时一致的行为，那么这个类、对象或方法就是线程安全的
* 线程安全问题都是由全局变量（成员变量）以及静态变量引起的
* 若每个线程中对全局变量、静态变量只有读操作，没有写操作，一般来说，这个全局变量是线程安全的。若有多个线程同时执行写操作，一般都需要考虑线程同步，否则就有可能影响线程安全

### 需要考虑线程安全的情况

* 访问共享的变量或资源，会有并发风险，比如对象的属性、静态变量、共享缓存、数据库等
* check-then-act操作：一个线程读取了一个共享数据，并在此基础上决定其下一个的操作
* 不同的数据之间存在捆绑关系的时候
* 我们使用其他类的时候，如果对方没有声明自己是线程安全的，那么大概率会存在并发问题

#### 栗子1：多个线程a++导致的线程安全问题

```java
public class SumError implements Runnable {
    private static SumError instance = new SumError();
    private int sum = 0;
    // 记录1-10000有哪一次是计算出错的，默认值为false
    private boolean[] marked = new boolean[20001];
    // 记录真正执行的次数
    private AtomicInteger execNums = new AtomicInteger();
    // 记录出错的次数
    private AtomicInteger wrongNums = new AtomicInteger();
    private CyclicBarrier cyclicBarrier1 = new CyclicBarrier(2);
    private CyclicBarrier cyclicBarrier2 = new CyclicBarrier(2);

    @Override
    public void run() {
        // 考虑第一次就发生错误的情况
        marked[0] = true;
        for (int index = 0; index < 10000; index++) {
            try {
                /*
                 防止线程1执行完了，线程2还未执行检测，
                 又执行线程1的情况，又被cpu切换sum++了一次，这样会影响线程2漏检测
                  */
                cyclicBarrier2.reset();
                cyclicBarrier1.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            sum++;
            try {
                /*
                 防止线程1在执行设置为true时，线程2把sum++了一次，
                 导致线程1检测的位置不对
                  */
                cyclicBarrier1.reset();
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            try {
                // 防止线程1还未检测，线程2把sum++了一次
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            // 执行次数增加
            execNums.incrementAndGet();
            /*
             检测当前要累加的值是否已经被累加过了，如果是的话，就是计算出错了
             synchronized保证检测的步骤和检测完设置为true的步骤是原子性的
             有可能出现sum++了，第二个线程也sum++，原本我要判断的sum位置就错位了,所以需要CycleBarrier
              */
            synchronized (instance) {
                if (marked[sum] && marked[sum - 1]) {
                    System.out.println("发生了错误:" + sum);
                    // 发生错误的次数增加
                    wrongNums.incrementAndGet();
                }
                // 操作过后，设置true
                marked[sum] = true;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {

        Thread thread1 = new Thread(instance);
        thread1.start();

        Thread thread2 = new Thread(instance);
        thread2.start();

        thread1.join();
        thread2.join();
        System.out.println("最终的结果是:" + instance.sum);
        System.out.println("执行的次数是:" + instance.execNums);
        System.out.println("出现错误的次数是:" + instance.wrongNums);

    }
}
```



### synchronized作用

* synchronized的作用是加锁，所有的synchronieed方法都会顺序执行（这里指占用cpu的顺序）
* synchronized保证在**同一时刻**最多只有**一个**线程执行该段代码
* synchronized能保证可见性和原子性，线程在执行结束前会把独立内存的值刷新回主内存，从而保证可见性

### synchronized的执行方式

* 首先尝试获得锁
* 如果获得锁，则执行synchronized的方法体内容
* 如果无法获得锁，则等待并且不断尝试去获得锁，一旦锁被释放，则多个线程会同时去尝试获得锁，造成锁竞争的问题

### synchronized的缺陷

* 效率低
  * 锁竞争问题，在高并发、线程数量高时会引起CPU占用居高不下，或者直接宕机
  * 锁的释放情况少，只有正常结束或抛出异常的时候才释放锁
  * 试图获得锁时不能设定超时时间
  * 不能中断一个正在试图获得锁的线程
* 不够灵活（读写锁更灵活）
  * 加锁和释放的时机单一，每个锁仅有单一的条件（某个对象），可能是不够的
* 无法知道是否成功获取到锁

### synchronized的两个用法：对象锁和类锁

* 对象锁：一个对象一个锁，多个对象之间不会发生锁竞争

  * 方法锁，默认锁对象为this当前实例对象

  * 同步代码块锁，自己指定任意对象(指定锁对象为this，相当于方法锁)

  * ```java
    public synchronized void method(){}
    // 等价
    public void method(){
        synchronized(this){}
    }
    ```

* 类锁：所有对象共享一把锁，存在锁竞争

  * synchronized加在静态方法
  * 同步代码块指定锁为Class对象

#### 栗子1：synchronized保证线程安全

创建3个线程共享一个ThreadDemo01对象，并对对象中的成员变量进行了写操作。肯定会引发线程安全问题。

```java
public class ThreadDemo01 implements Runnable {

    private int count = 0;

    @Override
    public void run() {
        count++;
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ">count=" + count);
    }

    public static void main(String[] args) {
        ThreadDemo01 threadDemo01 = new ThreadDemo01();
        for (int i = 0; i < 3; i++) {
            new Thread(threadDemo01, "t" + i).start();
        }
    }
}

```

输出结果：

存在线程安全问题！三条线程使用同一个threadDemo01创建线程。对threadDemo01对象中的count变量进行写的操作。

```java
t0>count=3
t1>count=3
t2>count=3
```

##### 使用synchronized方法加对象锁，解决线程安全问题：

```java
@Override
public synchronized void run() {
    count++;
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName() + ">count=" + count);
}
```

输出结果：

每隔2s中输出一次结果，不存在线程安全问题！

当t0执行run方法时，对threadDemo01对象加对象锁，当t1执行run方法时，必须等待t0释放锁才能执行。

所以sleep2s后，t0执行完，t1就可以执行了。

```java
t0>count=1
t2>count=2
t1>count=3
```

##### 使用同步代码块方法加对象锁，解决线程安全问题

```java
@Override
    public void run() {
        synchronized (this) {
            count++;
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ">count=" + count);
        }
    }
```

##### 使用同步代码块指定锁为Class对象加类锁，解决线程安全问题

```java
@Override
    public void run() {
        synchronized (ThreadDemo01.class) {
            count++;
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ">count=" + count);
        }
    }
```

##### 使用static变量加类锁，解决线程安全问题

```java
private static int count = 0;

    @Override
    public void run() {
        add();
    }

    public synchronized static void add() {
        count++;
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ">count=" + count);
    }
```

#### 栗子2：对象锁和类锁

```java
public class ThreadDemo03 {
    private int count = 0;

    //如果是static变量会怎样?
    public synchronized void add() {
        count++;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ">count=" + count);
    }

    public static void main(String[] args) {
        final ThreadDemo03 thread1 = new ThreadDemo03();
        final ThreadDemo03 thread2 = new ThreadDemo03();
        Thread t1 = new Thread(() -> thread1.add());
        Thread t2 = new Thread(() -> thread1.add());
        t1.start();
        t2.start();
    }

}
```

执行结果：

同一个对象同一把锁

```java
Thread-0>count=1
Thread-1>count=2
```

把main方法修改一下：

```java
public static void main(String[] args) {
    final ThreadDemo03 thread1 = new ThreadDemo03();
    final ThreadDemo03 thread2 = new ThreadDemo03();
    Thread t1 = new Thread(() -> thread1.add());
    Thread t2 = new Thread(() -> thread2.add());
    t1.start();
    t2.start();
}
```

执行结果：

不同对象，不同的锁

```java
Thread-0>count=1
Thread-1>count=1
```

如果把count变量升级为静态变量：

```java
public class ThreadDemo03 {
    private static int count = 0;

    //如果是static变量会怎样?
    public synchronized static void add() {
        count++;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ">count=" + count);
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> ThreadDemo03.add());
        Thread t2 = new Thread(() -> ThreadDemo03.add());
        t1.start();
        t2.start();
    }

}
```

执行结果：

对象锁升级为类锁，存在锁竞争问题

```java
Thread-0>count=1
Thread-1>count=2
```

### 对象锁的同步与异步

同步：必须等待方法执行完毕，才能向下执行，共享资源访问的时候，为了保证线程安全，必须同步

异步：不用等待其他方法执行完毕，即可以立即执行，例如Ajax异步

* 对象锁只针对synchronized修饰的方法生效、对象中的所有synchronized方法都会同步执行、而非 synchronized方法异步执行
* 避免误区：类中有两个synchronized方法，两个线程分别调用两个方法，相互之间也需要竞争锁， 因为两个方法从属于一个对象，而我们是在对象上加锁

#### 栗子1：同步与异步执行

```java
public class ThreadDemo02 {
    //同步执行
    public synchronized void print1() {
        System.out.println("print1执行,时间:" + new Date());
        System.out.println(Thread.currentThread().getName() + ">hello!");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    //异步执行
    public void print2() {
        System.out.println("print2执行,时间:" + new Date());
        System.out.println(Thread.currentThread().getName() + ">hello!");
    }

    public static void main(String[] args) {
        ThreadDemo02 thread = new ThreadDemo02();
        Thread t1 = new Thread(() -> thread.print1(), "thread1");
        Thread t2 = new Thread(() -> thread.print2(), "thread1");
        t1.start();
        t2.start();
    }

}

```

执行结果：

两个线程共享一个ThreadDemo02对象，当t1执行synchronized void print1()时会加对象锁，而此时t2执行public void print2()能不能执行成功？是不是要等待t1释放锁才能执行？

从运行结果来看，在同一时间，t1执行了print1方法，t2执行了print2方法，说明t2不需要等待t1释放锁。这时因为对象中的所有synchronized方法都会同步执行、而非 synchronized方法异步执行

```java
print2执行,时间:Thu Jun 25 18:26:43 CST 2020
print1执行,时间:Thu Jun 25 18:26:43 CST 2020
thread1>hello!
thread1>hello!
```

如果给print2也加上synchronized关键字：

```java
public synchronized void print2() {
    System.out.println("print2执行,时间:" + new Date());
    System.out.println(Thread.currentThread().getName() + ">hello!");
}
```

执行结果：

同一个对象的多个synchronized方法是同步执行的，多条线程如果调用的是synchronized方法，是要等待其他线程对该对象释放锁后才能执行的。

t1对print1加对象锁，t2要像访问同一对象的print2方法，就需要等待t1执行完毕释放锁才能继续执行，可以看到输出结果中sleep3s等待t1释放锁，才执行print2方法 

```java
print1执行,时间:Thu Jun 25 18:34:06 CST 2020
thread1>hello!
print1执行,时间:Thu Jun 25 18:34:09 CST 2020
thread1>hello!
```

### 脏读

由于同步和异步方法的执行个性，如果不从全局上进行并发设计很可能会引起数据的不一致，也就是所谓的脏读。

* 多个线程访问同一资源，在一个线程修改数据的过程中，有另外的线程来读取数据，就会引起脏读数据的产生。
* 为了避免脏读，我们一定要保证数据修改操作的原子性，并且对读操作也要进行同步控制

#### 栗子1：脏读的产生

线程修改ThreadDemo04的name和address属性，但是address还未修改成功，就被读取，从而发生了脏读。

```java
public class ThreadDemo04 {
    private String name = "张三";
    private String address = "大兴";

    public synchronized void setVal(String name, String address) {
        this.name = name;
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.address = address;
        System.out.println("setValue最终结果：username = " + name + " , address = " + address);
    }

    public void getVal() {
        System.out.println("getValue方法得到：username = " + this.name + " , address = " + this.address);
    }

    public static void main(String[] args) throws Exception {
        final ThreadDemo04 dr = new ThreadDemo04();
        Thread t1 = new Thread(() -> dr.setVal("李四", "昌平"));
        t1.start();

        // 睡眠1s,t1线程肯定还未执行结束
        Thread.sleep(1000);

        dr.getVal();
    }
}

```

执行结果：

线程还未将地址修改，main线程将旧的数据读取了出来。

```java
getValue方法得到：username = 李四 , address = 大兴
setValue最终结果：username = 李四 , address = 昌平
```

要解决脏读问题，对读操作也要进行同步控制：

```java
 public synchronized void getVal() {
        System.out.println("getValue方法得到：username = " + this.name + " , address = " + this.address);
    }
```

执行结果：

```java
setValue最终结果：username = 李四 , address = 昌平
getValue方法得到：username = 李四 , address = 昌平
```

###  synchronized的性质

> 不可中断性

一旦这个锁已经被别人获得了，如果我还想获得，我只能选择等待或者阻塞，直到别的线程释放这个锁。如果别人永远不释放锁，那么我只能永远地等下去。

> 可重入性

同一个线程得到了一个对象的锁之后，再次请求此对象时可以再次获得该对象的锁。（同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁）

1.优点：避免死锁，提升封装性

2.粒度：

* 证明同一个方法是可重入的（递归调用本方法）
* 证明可重入不要求是同一个方法（同一个对象内的多个synchromized方法可以锁重入）
* 证明可重入不要求是同一个类中的（父子类可以锁重入）

#### 栗子1：同一个对象内的多个synchronized方法

```java
public class ThreadDemo05 {
    public synchronized void run1(){
        System.out.println(Thread.currentThread().getName()+">run1...");
        //调用同类中的synchronized方法不会引起死锁
        run2();
    }

    public synchronized void run2(){
        System.out.println(Thread.currentThread().getName()+">run2...");
    }

    public static void main(String[] args) {
        final ThreadDemo05 threadDemo05 = new ThreadDemo05();
        Thread thread = new Thread(() -> threadDemo05.run1());
        thread.start();
    }
}

```

执行结果：

线程首先调用了run1()方法，对threadDemo05对象加对象锁，在run1()方法中又调用了synchronized修饰的run2()方法，按理说，run1()方法还没有执行完毕，没有释放锁，执行run2()方法没有获得锁，是会发生死锁的。但是运行结果说明，同个对象的synchronized方法是可以锁重入的。

```java
Thread-0>run1...
Thread-0>run2...
```

#### 栗子2 ：父子类的锁重入

在child类中synchronized方法中调用了Parent类的synchronized方法。

```java
class Parent {
    public int i = 10;

    public synchronized void runParent() {
        i--;
        System.out.println("Parent>>>>i=" + i);
    }
}

class Child extends Parent {

    public synchronized void runChild() {
        while (i > 0) {
            i--;
            System.out.println("Child>>>>i=" + i);
            //调用父类中的synchronized方法不会引起死锁
            this.runParent();
        }
    }
}

public class ThreadDemo06 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            Child sub = new Child();
            sub.runChild();
        });
        t1.start();
    }
}
```

执行结果：

Child类继承了Parent的i变量和runParent()。

当线程调用Child类的runChild()时，对sub对象加对象锁，而在runChild方法中又调用了runParent方法，按理说，runChild方法没有执行完毕，不会释放锁，runParent方法执行又需要获得锁，会发生死锁现象。从运行结果来看，父子类之间的多个synchronized方法是可以锁重入的。

```java
Child>>>>i=9
Parent>>>>i=8
Child>>>>i=7
Parent>>>>i=6
Child>>>>i=5
Parent>>>>i=4
Child>>>>i=3
Parent>>>>i=2
Child>>>>i=1
Parent>>>>i=0
```

### 抛出异常释放锁

一个线程在获得锁之后执行操作，发生错误抛出异常，则自动释放锁。

* 可以利用抛出异常，主动释放锁
* 程序异常时防止资源被死锁，无法释放
* 异常释放锁可能导致数据不一致

#### 栗子1：主动抛出异常，释放锁

在变量i加到10的时候主动释放锁，才能让get方法获取变量i的值

```java
public class ThreadDemo07 {
    private int i = 0;

    public synchronized void add() {
        while (true) {
            i++;
            System.out.println(Thread.currentThread().getName() + "-run>i=" + i);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if (i == 10) {
                throw new RuntimeException();
            }
        }
    }

    // 防止脏读
    public synchronized void get() {
        System.out.println(Thread.currentThread().getName() + "-get>i=" + i);
    }

    public static void main(String[] args) throws InterruptedException {
        final ThreadDemo07 threadDemo07 = new ThreadDemo07();
        new Thread(() -> threadDemo07.add(), "t1").start();
        // 保证t1线程先执行
        Thread.sleep(1000);
        new Thread(() -> threadDemo07.get(), "t2").start();
    }
}

```

执行结果：

线程调用add方法，在add方法中对i进行累加，本会一直无限循环进行累加，但在add方法中主动抛出了异常，释放锁，而get方法就可以运行得到i的值。

这里需要注意的是，即使add方法没有释放锁，get方法也可以锁重入。但因为synchronized方法是同步执行的，所以get方法必须等add方法执行完才能执行。

==**所以这里要区分死锁问题和同步执行问题**==

```java
t1-run>i=1
t1-run>i=2
t1-run>i=3
t1-run>i=4
t1-run>i=5
t1-run>i=6
t1-run>i=7
t1-run>i=8
t1-run>i=9
t1-run>i=10
t2-get>i=10
Exception in thread "t1" java.lang.RuntimeException
	at thread04.ThreadDemo07.run(ThreadDemo07.java:29)
	at thread04.ThreadDemo07.lambda$main$0(ThreadDemo07.java:41)
	at java.lang.Thread.run(Thread.java:748)
```

### synchronized代码块

可以达到更细粒度的控制

* 当前对象锁
* 类锁
* 任意对象锁

#### 栗子

```java
public class ThreadDemo08 {
    public void run1() {
        synchronized (this) {
            try {
                System.out.println(Thread.currentThread().getName() + ">当前对象锁..");
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void run2() {
        synchronized (ThreadDemo08.class) {
            try {
                System.out.println(Thread.currentThread().getName() + ">类锁..");
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private Object objectLock = new Object();

    public void run3() {
        synchronized (objectLock) {
            try {
                System.out.println(Thread.currentThread().getName() + ">任意对象锁..");
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    //测试方法
    public static void test(final int type) {
        if (type == 1) {
            System.out.println("当前对象锁测试...");
        } else if (type == 2) {
            System.out.println("类锁测试...");
        } else {
            System.out.println("任意对象锁测试...");
        }
        final ThreadDemo08 demo1 = new ThreadDemo08();
        final ThreadDemo08 demo2 = new ThreadDemo08();
        Thread t1 = new Thread(() -> {
            if (type == 1) {
                demo1.run1();
            } else if (type == 2) {
                demo1.run2();
            } else {
                demo1.run3();
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            if (type == 1) {
                demo1.run1();
            } else if (type == 2) {
                demo2.run2();
            } else {
                demo1.run3();
            }
        }, "t2");
        t1.start();
        t2.start();
    }
    
    public static void main(String[] args) {
        test(1);  // 测试当前对象锁
        //test(2);  // 测试类锁
        //test(3);  // 测试任意对象锁
    }
}
```

运行结果：test(1)

可以用同步代码块对当前对象加锁。

```java
当前对象锁测试...
t1>当前对象锁..
t2>当前对象锁..
```

test(2)：

可以用同步代码块对类加锁。

```java
类锁测试...
t1>类锁..
t2>类锁..
```

test(3):

可以用同步代码块对任意对象加锁。

```java
任意对象锁测试...
t1>任意对象锁..
t2>任意对象锁..
```

我们测试一下当类锁和对象锁同时存在的情况：

```java
 public static void main(String[] args) {
     final ThreadDemo08 demo1 = new ThreadDemo08();
     final ThreadDemo08 demo2 = new ThreadDemo08();
     System.out.println(new Date());
     // 加类锁
     Thread t1 = new Thread(() -> demo1.run2(), "t1");
     t1.start();

     // 加对象锁
     Thread t2 = new Thread(() -> demo2.run1(), "t2");
     t2.start();
     System.out.println(new Date());
}
```

执行结果：

从运行结果可以看到，在同一时间，两条线程并行执行了。并没有sleep2s，也就是说对象锁并不需要等待类锁释放才执行。

```java
Thu Jun 25 22:19:14 CST 2020
t1>类锁..
Thu Jun 25 22:19:14 CST 2020
t2>当前对象锁..
```

总结：**同类型锁之间互坼，不同类型的锁之间互不干扰**

### synchronized使用的几种情况

1. 两个线程同时访问**同一对象**的synchronized方法
   * 同一时刻只有一个线程访问，另一个线程阻塞，对象锁
2. 两个线程同时访问**不同对象**的synchronized方法
   * 两个线程并行执行，互不影响
3. 两个线程访问的是synchronized的static方法
   * 同一时刻只有一个线程访问，另一个线程阻塞，类锁
4. 同时访问synchronized修饰的方法与没有sychronized修饰的方法
   * synchronized修饰的方法同步执行，没有synchronized修饰的方法异步执行
5. 访问同一个对象实例的不同的非static的synchronized方法
   * 同步执行，因为synchronized修饰的非static方法默认使用synchronized(this)加对象锁的方式
6. 同时访问**静态**synchronized方法与**非静态**的synchronized方法
   * 不会影响。同种类型的锁会互坼，不同类型的锁互不干扰。因为静态synchronized方法对class加锁，非静态synchronized方法对this加锁，不是同一把锁
7. 方法抛出异常，释放锁

总结(3个核心思想)：

* 一把锁只能同时被一个线程获取，没有拿到锁的线程必须等待（对应1、5的情况）
* 每个实例都对应自己的一把锁，不同实例之间互不影响。例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象共用同一把类锁（对应2、3、4、6的情况）
* 无论是方法正常执行完毕或者方法抛出异常，都会释放锁（对应第7种情况）

### synchronized原理

> 加锁和释放锁的原理

```java
public synchronized void method() {}

// synchronized加锁的机制类似如下代码

Lock lock = new ReentrantLock();
public void method() {
    lock.lock();
    // 情况1：抛出异常释放锁的情况
    try {

    } finally {
        lock.unlock();
    }

    // 情况2：正常结束释放锁的情况
    lock.unlock();
}
```

> 反编译查看monitor

javap -verbose  SynchronizedLock.class 查看反编译文件。

一个monitorenter可以对应多个monitorexit，是因为进入之后度与退出的情况并不是一一对应的，多种退出方式使得exit数量可能大于enter的数量。

monitorenter使monitor计数器+1，monitorexit使计数器-1,如果变成没有变成0，说明之前是重入的，那么线程继续持有锁

```java
Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #3                  // Field obj:Ljava/lang/Object;
         4: dup
         5: astore_1
         6: monitorenter	// 重点
         7: aload_1
         8: monitorexit		// 重点
         9: goto          17
        12: astore_2
        13: aload_1
        14: monitorexit		// 可能存在一个monitorenter对应着多个monitorexit,因为存在两种退出情况
        15: aload_2
        16: athrow
        17: return
```

### 锁失效问题

* 不要在线程中修改对象锁的引用，引用被改变会导致锁失效。
* 在线程中修改了锁对象的属性，而不修改引用则不会引起锁失效，不会产生线程安全问题

#### 栗子1：修改对象的引用会引起锁失效

```java
public class ThreadDemo09 {
    private String lock = "lock handler";

    private void method() {
        synchronized (lock) {
            try {
                System.out.println("当前线程 : " + Thread.currentThread().getName() + "开始");
                Thread.sleep(2000);
                System.out.println("当前线程 : " + Thread.currentThread().getName() + "结束");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {

        final ThreadDemo09 changeLock = new ThreadDemo09();
        Thread t1 = new Thread(() -> changeLock.method(), "t1");
        Thread t2 = new Thread(() -> changeLock.method(), "t2");
        t1.start();
        t2.start();
    }
}
```

执行结果：

正常的执行结果：

```java
当前线程 : t1开始
当前线程 : t1结束
当前线程 : t2开始
当前线程 : t2结束
```

由于锁的引用被改变，所以t2线程也进入到method方法内执行。

```java
private void method() {
        synchronized (lock) {
            try {
                System.out.println("当前线程 : " + Thread.currentThread().getName() + "开始");
                //锁的引用被改变,则其他线程可获得锁，导致并发问题
                lock = "change lock handler";
                Thread.sleep(2000);
                System.out.println("当前线程 : " + Thread.currentThread().getName() + "结束");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
```

执行结果：

从运行结果来看，synchronized同步代码块不能保证原子性了，同步失效了。这是因为在线程执行过程中，线程A修改了锁的引用，则线程B实际上得到了新的对象锁，而不是锁被释放了，因此引发了线程安全问题。

```java
当前线程 : t1开始
当前线程 : t1结束
当前线程 : t2开始
当前线程 : t2结束
```

#### 栗子2：修改对象属性不会引起锁失效

```java
class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "DemoThread10 [name=" + name + ", age=" + age + "]";
    }
}

public class ThreadDemo10 {
    private Person person = new Person();

    public void changeUser(String name, int age) {
        synchronized (person) {
            System.out.println("线程" + Thread.currentThread().getName() + "开始" + person);
            // 修改对象的属性，不会引起锁失效
            person.setAge(age);
            person.setName(name);
            System.out.println("线程" + Thread.currentThread().getName() + "修改为" + person);
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" + Thread.currentThread().getName() + "结束" + person);
        }
    }

    public static void main(String[] args) {
        final ThreadDemo10 thread10 = new ThreadDemo10();
        new Thread(() -> thread10.changeUser("小白", 99), "t1").start();
        new Thread(() -> thread10.changeUser("小黑", 100), "t2").start();
    }
}
```

执行结果：

synchronized加的对象锁没有失效，没有出现线程安全问题。

```java
线程t1开始DemoThread10 [name=null, age=0]
线程t1修改为DemoThread10 [name=小白, age=99]
线程t1结束DemoThread10 [name=小白, age=99]
线程t2开始DemoThread10 [name=小白, age=99]
线程t2修改为DemoThread10 [name=小黑, age=100]
线程t2结束DemoThread10 [name=小黑, age=100]
```

### 一句话总结Synchronized

JVM会自动通过使用monitor来加锁和解锁，保证了同一时刻只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。



