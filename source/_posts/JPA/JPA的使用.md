---
title: JPA的使用.md
date: 2020-10-06 15:12:35
categories: JPA
tags: JPA
---



### JPA查询的使用

1. 创建一个接口，实现JpaRepository<T,R>接口。T是操作的实体，R是操作的实体的主键的类型



### 查询所有数据

```java
findAll()
```



### 分页

PageRequest.of构造Pageable分页对象：

* pageNumber：页码
* size：每页条数
* Sort：排序对象，有多个重载方法，descending为倒叙排序

findAll方法可以接收一个分页对象Pageable进行查询，查询结果返回Page<T>

```java
public Page<Spu> getLatestPagingSpu(Integer pageNumber, Integer size) {
    Pageable page = PageRequest.of(pageNumber, size, Sort.by("createTime").descending());
    return spuRepository.findAll(page);
}
```

Page<T>包含了很多分页的信息，比如分页大小，数据总数，当前页码等，可以用一个实体类封装这些分页信息，并返回给前端：

```java
public class Paging<T> {
    /**
     * 数据总数
     */
    private Long total;
    /**
     * 分页大小
     */
    private Integer count;
    /**
     * 当前分页
     */
    private Integer pageNumber;
    /**
     * 分页总数
     */
    private Integer totalPage;
    /**
     * 分页数据
     */
    private List<T> items;

    public Paging(Page<T> page) {
        this.initPageParams(page);
        this.items = page.getContent();
    }

    protected void initPageParams(Page<T> page) {
        this.total = page.getTotalElements();
        this.count = page.getSize();
        this.pageNumber = page.getNumber();
        this.totalPage = page.getTotalPages();
    }
}
```

有时候返回前端的是“简化版”的vo，比如JPA返回的分页对象Page<Spu>，而需要返回给前端的是Pageing<SimplifySpu>，就需要将Spu的属性复制到SimplifySpu中，我们可以写一个可以复制对象属性的PagingDozer<Spu,SimplifySpu>类：

```java
/**
 * @Author Ming
 * @Date 2020/06/12 22:54
 * @Description 可复制属性的分页对象
 */
public class PagingDozer<T, R> extends Paging {

    private Mapper mapper = DozerBeanMapperBuilder.buildDefault();
    
    @SuppressWarnings("unchecked")
    public PagingDozer(Page<T> page, Class<R> clazz) {
        super.initPageParams(page);

        List<T> items = page.getContent();
        List<R> voList = items.stream()
                .map(item -> mapper.map(item, clazz))
                .collect(Collectors.toList());
        this.setItems(voList);
    }
}
```



### 高级查询

#### 原生SQL查询

设置nativeQuery=true，使用原生的sql查询

```java
@Query(nativeQuery = true,value = "select coupon.* from coupon inner join coupon_category on coupon.id = coupon_category.coupon_id inner join category on coupon_category.category_id = category.id")
    Optional<List<Coupon>> findByCategory(Long categoryId, Date now);
```

#### JPQL

使用实体代替数据表，参数通过“：参数名”的格式传递。

如果配置配置导航属性，需要使用ON写连接条件。

**注意，必须使用别名。**

```java
 @Query(value = "select c from Coupon c\n" +
            "join c.categoryList ca\n" +
            "join Activity a on a.id = c.activityId \n" +
            "where ca.id = :categoryId \n" +
            "and a.startTime < :now \n" +
            "and a.endTime > :now")
 Optional<List<Coupon>> findByCategory(Long categoryId, Date now);
```

生成的sql语句：

```sql
select
        coupon0_.id as id1_4_,
        coupon0_.create_time as create_t2_4_,
        coupon0_.delete_time as delete_t3_4_,
        coupon0_.update_time as update_t4_4_,
        coupon0_.activity_id as activity5_4_,
        coupon0_.description as descript6_4_,
        coupon0_.end_time as end_time7_4_,
        coupon0_.full_money as full_mon8_4_,
        coupon0_.minus as minus9_4_,
        coupon0_.rate as rate10_4_,
        coupon0_.remark as remark11_4_,
        coupon0_.start_time as start_t12_4_,
        coupon0_.title as title13_4_,
        coupon0_.type as type14_4_,
        coupon0_.valitiy as valitiy15_4_,
        coupon0_.whole_store as whole_s16_4_ 
    from
        coupon coupon0_ 
    inner join
        coupon_category categoryli1_ 
            on coupon0_.id=categoryli1_.coupon_id 
    inner join
        category category2_ 
            on categoryli1_.category_id=category2_.id 
            and (
                category2_.delete_time is null 
                and category2_.online = 1
            ) 
    inner join
        activity activity3_ 
            on (
                activity3_.id=coupon0_.activity_id
            ) 
    where
        category2_.id=? 
        and activity3_.start_time<? 
        and activity3_.end_time>?

```

