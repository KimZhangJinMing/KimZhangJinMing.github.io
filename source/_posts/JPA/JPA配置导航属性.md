---
title: JPA配置导航属性.md
date: 2020-10-04 14:57:35
categories: 数据库
tags: JPA
---

### SpringBoot使用JPA配置

1. 添加maven依赖：data-jpa、starter-jdbc、mysql-connector-java

2. 配置JPA

   ![jpa配置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201004164138673.png)

### JPA常用注解

* @Entity：表示一个实体类
* @Id：设置主键
* @Table：在实体类上使用，设置数据表名称
* @Transient：在属性中存在，但不映射到数据表
* @OneToMany：一对多的关系，可以通过fetch属性来设置懒加载、急加载，默认是懒加载
* @ManyToOne：多对一的关系
* @JoinColumn(name="")：设置外键
* @GeneratedValue(strategy = GenerationType.IDENTITY)：自增长
* @ManyToMany：多对多的关系
* @JoinTable(name = "theme_spu",joinColumns = @JoinColumn(name = "theme_id"),
      inverseJoinColumns = @JoinColumn(name = "spu_id"))：设置中间表的属性
* @ForeignKey(name ="null")：禁止生成物理外键



### 单向一对多的配置

假设有两个实体类Banner和BannerItem，它们是一对多的关系。

实体类Banner的定义如下：

```java
@Entity
public class Banner {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```

实体类BannerItem的定义如下：

```java
@Entity
public class BannerItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Integer type;
}
```

如果想配置Banner类中有多个BannerItem这种单向一对多的关系，需要在Banner类中添加属性：List<BannerItem> bannerItems，并使用注解@OneToMany标记在bannerItems类上，表示一对多的关系。

```java
@Entity
public class Banner {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany
    private List<BannerItem> bannerItems;
}
```

但是，实际运行会发现，数据库生成了3张表，banner，banner_item，banner_banner_items，其中banner_banner_items是中间表，用来维护一对多的这种关系。

其实，一对多的关系并不需要生成一张中间表来维护。我们就需要告诉JPA，在生成数据表的时候，不要给我生成中间表，此时，就需要使用@JoinColumn来指明外键了。

```java
@OneToMany
@JoinColumn(name = "banner_id")
private List<BannerItem> bannerItems;
```

这样，就只会生成banner和banner_item表，且在banner_item中自动生成了一个banner_id字段来表示外键关系，通过查看查看外键，也证明了banner_id存在物理外键。



### 双向一对多的配置

有时候，业务有双向一对多的场景。比如，查询Banner的时候，要查询出对应的BannerItem。查询BannerItem的时候，也需要知道BannerItem是属于哪个Banner的。

双向一对多需要在两个实体类都加上注解，@OneToMany和@ManyToOne。

Banner：

```java
@Entity
public class Banner {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "banner")
    private List<BannerItem> bannerItems;
}
```

BannerItem：

```java
@Entity
public class BannerItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Integer type;

    @ManyToOne
    @JoinColumn(name = "banner_id")
    private Banner banner;
}
```

我们注意到一个细节：Banner类上@OneToMany多了一个mappedBy属性，值为BannerItem中的属性banner。@JoinColumn不再标记在Banner中，而是标记在了BannerItem中，这是为什么呢？

这里，要提出两个概念：

* 关系维护方(多方)
* 关系被维护端(一方)

在双向多对多的这种关系中，关系维护方是在多方，也就是BannerItem中，所以@JoinColumn需要标记在关系维护方中。

**在双向一对多的关系中，@JoinColumn也可以不写，但是mappedBy必须要写。**



#### 问题：JPA自动生成了banner_id的外键，我们还可以在BannerItem中定义多一个BannerId的属性吗？

运行之后，会报错，错误显示有重复的banner_id字段。

```java
Table [banner_item] contains physical column name [banner_id] referred to by multiple logical column names: [banner_id], [bannerId]
```

经过测试，有一种办法可以在BannerItem中定义BannerId属性，又不影响JPA自动生成的banner_id外键：

在关系维护端的@JoinColumn添加@JoinColumn(name = "banner_id",insertable = false,updatable = false)，

但是属性名称必须和@JoinColumn的name属性相同。

```java
@Entity
public class BannerItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Integer type;

    private Long banner_id;

    @ManyToOne
    @JoinColumn(name = "banner_id",insertable = false,updatable = false)
    private Banner banner;
}
```



### 单向多对多的配置

假设有两个实体类Theme和Spu，它们是一对多的关系。

实体类Theme：

```java
@Entity
public class Theme {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(name = "theme_spu",joinColumns = @JoinColumn(name = "theme_id"),
    inverseJoinColumns = @JoinColumn(name = "spu_id"))
    private List<Spu> spuList;
}
```

实体类Spu：

```java
@Entity
public class Spu {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```

同单向一对多的配置差不多，单向多对多会生成一种中间表来维护关系。@JoinTable是用来指定生成的中间表的属性的。比如name指明了中间表的表名为theme_spu。



### 双向多对多的配置

只需要在Spu中添加@ManyToMany注解，并设置mapperBy属性即可。

**需要注意的是，同双向一对多的关系不同的是，双向多对多的关系维护方并不是固定的。**比如这里的关系维护方是Theme类，因为@JoinTable注解标记在Theme类中，但也可以标记在Spu类中，使Spu成为关系维护方。

**对于查询数据来说，谁是关系维护方并不影响，因为都会查询出关联的数据。**

```java
@Entity
public class Spu {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "spuList")
    private List<Theme> themeList;
}
```



### 如何禁止JPA生成物理外键？

* 在@JoinColumn或者@JoinTable的属性中添加foreignKey

  ```java
  @JoinTable(foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
  ```

* 使用过时的@ForeignKey注解

  ```java
  @org.hibernate.annotations.ForeignKey(name = "null")
  ```

* 重写方言Dialet



### 生成一个父类，不映射成数据表

标记上@MappedSuperclass表示不映射成数据表，只是作为@Entity的父类。

```java
/**
 * @Author Ming
 * @Date 2020/06/07 14:04
 * @Description 标记为abstract,BaseEntity不能单独被new
 *              JsonIgnore表示在序列化的时候忽略,不返回给前端
 *              MappedSuperclass表示不映射成数据表,只是作为Entity的父类
 */
@Getter
@Setter
@MappedSuperclass
public abstract class BaseEntity {
    @JsonIgnore
    private Date createTime;
    @JsonIgnore
    private Date updateTime;
    @JsonIgnore
    private Date deleteTime;
}
```

