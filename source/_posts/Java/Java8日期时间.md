---
title: Java8日期时间.md
date: 2020-10-18 22:10:35
categories: 基础
tags: Java8
---

### LocalDate、LocalTime

LocalDate是一个不可变对象，它只提供了简单的日期，并不含当天的时间信息。另外，它也不附带任何与时区相关的信息。

* 使用静态方法of创建
* 通过解析代表它们的字符串来创建，一旦传递的字符串参数无法被解析为合法的LocalDate或LocalTime对象，会抛出DateTimeParseException。
* 多种方法获取年份、月份、星期几等。TemporalField是一个接口，它定义了如何访问temporal对象某个字段的值。ChronoField枚举实现了这一接口，可以很方便地使用get方法得到枚举元素的值。

```java
LocalDate localDate = LocalDate.of(2020, 1, 15);
System.out.println(localDate.getYear()); // 2020
System.out.println(localDate.get(ChronoField.YEAR)); // 2020

Month month = localDate.getMonth();
System.out.println(localDate.get(ChronoField.MONTH_OF_YEAR));// 1
System.out.println(month.getLong(ChronoField.MONTH_OF_YEAR));// 1
System.out.println(month.getValue());// 1
System.out.println(localDate.lengthOfMonth());// 31

System.out.println(localDate.getDayOfYear());// 15
System.out.println(localDate.getDayOfMonth());// 15
DayOfWeek dayOfWeek = localDate.getDayOfWeek();
System.out.println(dayOfWeek.getValue());// 3
System.out.println(dayOfWeek.getLong(ChronoField.DAY_OF_WEEK));// 3

```

如果使用Parse方法解析的字符串是不带毫秒，纳秒的，获得毫秒、纳秒的值为0.

```java
LocalTime localTime = LocalTime.parse("22:17:32");
//LocalTime localTime = LocalTime.now();
System.out.println(localTime.getHour());
System.out.println(localTime.getLong(ChronoField.MILLI_OF_SECOND));// 毫秒
System.out.println(localTime.getLong(ChronoField.MICRO_OF_SECOND));// 纳秒
```



### LocalDataTime和LocalDate、LocalTime的转换

* LocalDateTime可以通过静态方法of创建，也可以通过传入LocalDate、LocalTime创建
* LocalDateTime可以通过LocalTime传递LocalDate对象来创建，也可以通过LocalDate传递LocalTime对象来创建
* LocalDateTime可以通过toLocalDate()、toLocalTime()方法来转换成LocalDate、LocalTime对象

```java
LocalDate localDate = LocalDate.now();
LocalTime localTime = LocalTime.now();
LocalDateTime localDateTime1 = LocalDateTime.of(localDate, localTime);
LocalDateTime localDateTime2 = localDate.atTime(22, 25, 32);
LocalDateTime localDateTime3 = localTime.atDate(localDate);

LocalDate localDate1 = localDateTime1.toLocalDate();
LocalTime localTime1 = localDateTime1.toLocalTime();
System.out.println(localDateTime1);// 2020-10-18T22:27:28.825
System.out.println(localDateTime2);// 2020-10-18T22:25:32
System.out.println(localDateTime3);// 2020-10-18T22:27:28.825
```

### Instant

Instant的设计初衷是为了便于机器使用。它包含的是由秒和纳秒所构成的数字，它无法处理哪些我们非常容易理解的时间单位，但是可以通过Duration和Period类使用Instant。

### Duration和Period

Duration是计算两个LocalTime、LocalDateTime、Instant之间的间隔时间的。

Period是计算两个LocalDate的间隔日期的。

它们都可以通过静态方法between来创建。

**注意：由于LocalDateTime和Instant是为了不同的目的而设计的，一个是为了便于人阅读使用，一个是便于机器处理，所以不能讲二者混用，会抛出DateTimeException。**



### 克隆对象

如果你已经有了一个LocalDate对象，想要创建它的一个修改版，最直接的方法是使用withAttribute方法。withAttribute方法会创建对象的一个副本，并按照需要修改它的属性。也可以采用更加通用的with方法。

```java
LocalDate localDate = LocalDate.now();// 2020-10-18
LocalDate localDate1 = localDate.withYear(2011);// 2011-10-18
LocalDate localDate2 = localDate.withDayOfMonth(25);// 2020-10-25
LocalDate localDate3 = localDate.with(ChronoField.MONTH_OF_YEAR, 9);// 2020-09-18
```

### 日期时间的计算

Temporal接口提供了很多的方法进行日期时间的计算，比如with，get等。所有的日期和时间API类都实现了这两个方法。

```java
LocalDate localDate = LocalDate.now();// 2020-10-18
LocalDate localDate1 = localDate.plusDays(3);// 2020-10-21
LocalDate localDate2 = localDate.minusYears(1);// 2019-10-18
LocalDate localDate3 = localDate.plus(6, ChronoUnit.MONTHS);// 2021-04-18
```

有时候，需要将日期调整到下个周日，下个工作日或者是本月的最后一天，可以使用重载的with方法，向其传递一个提供了更多定制化选择的TemporalAdjuster对象，更加灵活得处理日期。

日期和时间API已经提供了大量预定义的TemporalAdjusters，可以通过静态工厂方法访问它们。也可以通过实现TemporalAdjuster接口自定义。

```java
LocalDate localDate = LocalDate.now();// 2020-10-18
// 离现在最近的星期天 2020-10-18
LocalDate localDate1 = localDate.with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY));
// 这个月的最后一天 2020-10-31
LocalDate localDate2 = localDate.with(TemporalAdjusters.lastDayOfMonth());
```

