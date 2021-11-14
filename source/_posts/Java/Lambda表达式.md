---
title: Lambda.md
date: 2020-08-10 22:46:35
categories: 基础
tags: Lambda
---

### Java 8 新特性

* Lambda表达式，代码更少
* Stream API
* Optional，最大化减少空指针异常
* 便于并行
* Nashom引擎，允许在JVM上运行JS应用

### Lambda表达式

一段可以传递行为的代码....

举个栗子：

1.不使用Lambda表达式：

匿名类...注意需要加上；结尾

```java
public void test3(){
    Comparator<Integer> c = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1,o2);
        }
    };

    int result = c.compare(12, 22);
    System.out.println(result);
}		
```

2.使用Lambda表达式：

```java
public void test4(){
    Comparator<Integer> c = (o1,o2) -> Integer.compare(o1,o2);
    int result = c.compare(12, 22);
    System.out.println(result);
}
```

3.使用函数引用：

```java
public void test5(){
    Comparator<Integer> c =  Integer::compare;
    int result = c.compare(12, 22);
    System.out.println(result);
}
```

### Lambda表达式的使用

 * 本质：函数式接口的实例对象。	
   	* 函数式接口：只有一个抽象方法的接口
    * 函数式接口才能使用Lambda表达式

* 格式

  ->  lambda操作符

  (o1,o2)：形参列表，其实就是接口中的抽象方法的形参列表)

  右边：抽象方法的方法体

* 使用

  > 无参，无返回值

  

  ```java
  () -> {
      System.out.println("测试Lambda");
  }
  ```

  	>有且只有一个参数，无返回值

  

  ```java
  Consumer<String> -> (String s) -> {
      System.out.println(s);
  }
  ```

  	>数据类型可以省略，由编译器推断出来

  

  ```java
  Consumer<String> -> (s) -> {
      System.out.println(s);
  }
  ```

  > 若只有一个参数，参数的小括号可以省略

  

  ```java
  Consumer<String> -> s -> {
      System.out.println(s);
  }
  ```

  >两个以上的参数，多条执行语句，有返回值

  

  ```java
  Comparator<Integer> c = (o1,o2) -> {
  	System.out.println(o1);
  	System.out.println(o2);
  	return o1.compareTo(o2);
  }
  ```

  > 当Lambda体只有一条执行语句，return 与 {} 都可以省略

  ​	

  ```java
  (o1,o2) -> o1.compareTo(o2);
  ```

 * 总结：

    * Lambda的参数类型可以省略，由编译器推断，但是不一定准确(左边）
    * 形参有且仅有一个时，()可以省略(左边）
    * 只有一条执行语句，return 和 {} 可以省略(右边）

   ***

   ### 函数式接口

   * 可以在函数式接口上加上@FunctionalInterface注解，这样做可以检查它是不是函数接口
   * Lambda表达式就是一个函数式接口的实例
   * 以前的匿名实现类现在都可以使用Lambda表达式来写

   ***

   #### Java 内置的四大函数式接口

   * Consumer<T>：消费性接口，对参数类型为T的对象进行操作，不返回结果。包含方法void accept(T t)
   * Supplier<T>：供给型接口，返回类型为T的对象，不接收参数。包含方法T get()
   * Function<T,K>：函数型接口，可以接收参数，可以有返回值。对参数类型为T的对象进行操作，返回结果为K类型的对象。包含方法K apply(T t)
   * Predicate<T>：断定型接口，判断T类型的参数是否满足某约束，并返回boolean值，包含方法boolean test(T t)

   ***

   ### 函数式接口举例子

   定义一个过滤字符串的方法，该方法传入字符串List和Predicate接口，利用Predicate接口的test()方法判断是否满足某种规则，满足则返回。

   ```java
   public List<String> filterStrings(List<String> strs, Predicate<String> pre){
           List<String> list = new ArrayList<>();
           strs.forEach(s -> {
               if(pre.test(s)){
                   list.add(s);
               }
           });
           return list;
       }
   ```

   

   具体的判断规则，在调用的时候传递进去：

   ```java
   public void test8(){
           List<String> strings = Arrays.asList("北京", "天津", "南京");
           List<String> list = filterStrings(strings, s -> s.contains("京"));
           System.out.println(list);
       }
   ```

   ***

   ### 方法引用

   * 当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用
   * 要求：实现接口的抽象方法的参数列表和返回值，必须和方法引用的参数列表的和返回值类型保持一致
   * 格式：使用操作符：：将类(对象) 与 方法名分隔开来
   * 主要使用形式：
     * 对象：：实例方法
     * 类：：静态方法
     * 类：：实例方法

   
