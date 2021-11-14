### 1.redis是什么？

> 讲redis不得不讲nosql

* nosql不使用sql作为查询语言。
* nosql存储数据可以不需要固定的表格形式，它是基于键值对存储的。

>redis的定义

* redis可以用作数据库、缓存、消息中间件。
* redis支持多种类型的数据结构。如字符串(String),散列(hashes)、列表(lists)、集合(sets)、有序集合(sorted sets)。

### 2.redis,memcached,mysql的比较

#### 2.1 数据类型，存储方式的比较

| 数据库类型 | 数据存储方式    | 特色功能         |
| ---------- | --------------- | ---------------- |
| mysql      | 硬盘持久化      | 事务             |
| memcached  | 内存            |                  |
| redis      | 硬盘持久化+内存 | 支持多种数据类型 |

#### 2.2 内存缓存memcached、redis的比较

|           | 内存管理机制                                                 | 持久化方案                                                   | 缓存数据过期机制                                             | 支持的数据类型               |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| memcached | 基于Slab Allocation机制管理内存,其主要思想是按照预先规定的大小,将分配的内存分割成特定长度的块以存储相应长度的key-value数据记录,以完全解决内存碎片问题。`通过空闲列表判断存储状态。`[类似于Java虚拟机对象的分配,空闲列表] | 不支持数据持久化操作,所有的数据都保存在内存中                | 不会监视数据是否过期,而在获取数据时才会检查数据是否已经失效。[类似与懒加载] | 只支持单一的KV键值对数据类型 |
| redis     | 现场申请内存的机制,由于分配内存是连续的,并且很少使用空闲列表来优化内存分配,`会在一定程度上存在内存碎片`。[类似于Java虚拟机对象的分配,直接内存分配] | 支持rdb和aof两种不同的持久化方法。`rdb属于全量数据备份,备份的是数据。aof数据增量备份，备份的是指令。` | 定时、定期等多种缓存失效策略                                 | 支持五种数据类型             |



### 3.redis作为数据库和作为缓存的选择





### 4.redis设置

#### 4.1 远程连接设置

> 1.修改redis.conf配置文件

```shell
# 指定可以访问redis的网段。默认值是bind 127.0.0.1,只允许本地访问。
# bind ip,即只允许ip指定的地址能远程访问redis，连localhost都无法访问
# bind 0.0.0.0 或注释掉这一行,即允许任何ip访问
bind 0.0.0.0

# 保护模式，默认是protected-mode yes开启保护模式的
# 如果设置了bind或者requirepass,则可以设置为yes,不影响远程访问
protected-mode no

# 远程访问的密码,可以设置,也可以不设置,为了安全性,最好设置上密码
requirepass root
```

> 2.检查防火墙

```shell
# 检查6379端口是否开放
firewall-cmd --query-port=6379/tcp
# 永久开放6379端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
# 查看防火墙状态：
firewall-cmd --state 
# 启动防火墙
systemctl start firewalld
# 关闭防火墙
systemctl stop firewalld
# 检查防火墙开放的端口
firewall-cmd --permanent --zone=public --list-ports
# 重启防火墙
firewall-cmd --reload
# 防火墙开机自启动
systemctl enable firewalld.service
# 防火墙取消某一开放端口
firewall-cmd --zone=public --remove-port=9200/tcp --permanent
# 禁止firewall开机启动
systemctl disable firewalld.service 
```

> 3.如果使用阿里云,记得开放阿里云安全组的端口



#### 4.2 脚本启动redis认证密码

在使用脚本启动redis的时候，如果设置了redis密码，在进行`service redisd stop`操作的时候会出现以下错误:

```shell
(error) NOAUTH Authentication required
```

此时，可以在/etc/init.d/redisd脚本文件中添加-a密码选项。

```shell
# 添加-a 密码选项
$CLIEXEC  -a 'password' -p $REDISPORT shutdown
```



### 5.redis事务

#### 5.1 MULTI 与 EXEC

MULTI