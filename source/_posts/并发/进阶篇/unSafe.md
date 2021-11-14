---
title: Unsafe.md
date: 2020-08-10 22:46:35
categories: 并发
tags: JUC
---



### Unsafe类

* AtomicXX类大量采用Unsafe类完成底层操作
* 位于JDK的rt.jar包中，由BootstrapClassLoader加载的核心类
* Unsafe类中的方法几乎都为native方法
* 单例模式

#### 栗子1：直接通过内存地址为对象的属性赋值

```java
public class UnsafeDemo0 {

    private int age;

    public int getAge() {
        return age;
    }

    public static void main(String[] args) {
        UnsafeDemo0 demo0 = new UnsafeDemo0();

        // 获取Unsafe类实例,单例模式
        Unsafe unsafe = Unsafe.getUnsafe();
        try {
            //获取age属性的内存偏移地址，需要通过反射
            long ageOffset = unsafe.objectFieldOffset(UnsafeDemo0.class.getDeclaredField("age"));
            //设置age的值为11
            unsafe.putInt(demo0, ageOffset, 11);
            //输出结果
            System.out.println(demo0.getAge());
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}

```

执行结果：

执行结果抛出异常，只有BootstrapClassLoader加载的类才能调用。

```java
Exception in thread "main" java.lang.SecurityException: Unsafe
	at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
	at thread06.UnsafeDemo0.main(UnsafeDemo0.java:20)
```

#### 栗子2： 通过反射模式可以突破Unsafe类的安全限制 