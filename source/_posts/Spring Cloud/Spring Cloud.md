[TOC]



### 微服务架构理论入门

#### 微服务概念

* 提倡将单一应用程序拆分成一组小的服务

* 每个服务都是一个SpringBoot应用

#### SpringCloud概念

* 分布式微服务架构的一站式解决方案

#### 版本选型

**由SpringCloud版本决定SpringBoot版本。**

##### SpringBoot版本选型

* springBoot官方强烈建议使用2.0以上版本
* 2.2.2.RELEASE

##### SpringCloud版本选型

* springCloud H版本对应SpringBoot2.2.x版本

* Hoxton.SR1
* Clond Alibaba 2.1.0.RELEASE

### 停更的影响

* 服务注册中心
  * Eureka(×)
  * Zookeeper(✔)
  * Consul(✔)
  * Nacos(✔)
* 服务调用
  * Ribbon
  * LoadBalancer(✔)
  * Feign(x)
  * OpenFeign(✔)
* 服务降级
  * Hystrix(x)
  * Reslience4j(✔)
  * Sentinenl(✔ 国内使用)
* 服务网关
  * Zuul(x)
  * Gateway(✔)
* 服务配置
  * Config(x)
  * Nacos(✔)
* 服务总线
  * Bus(x)
  * Nacos(✔)



### 工程搭建

```xml
<packaging>pom</packaging>
```

#### dependencyManagement

* 出现在最顶层的父POM文件中
* 锁定版本，子POM不需要指定版本号就可以使用父POM的版本号，子POM不需要写groupid和version
* 如果子POM指定了具体的版本号，使用子POM的版本号
* 只是声明依赖，并不引入，子POM需要显示声明需要用的依赖 

#### 基础pom.xml

```xml
<dependencies>
    <!-- SpringCloud alibaba nacos-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--mysql-connector-java-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--jdbc-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <!--热部署-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



#### 微服务模块创建

* 创建module(创建完成回到父工程POM查看变化)
* 修改pom.xml
* 写配置文件yml
* 写启动类
* 写业务类

#### mybatis mapper文件模板

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sise.cloud.dao.StorageDao">
</mapper>
```


#### 构建时的几个小坑

* 使用mybatis时，可以在启动类上标记注解@MapperScan来注入Dao，也可以直接在Dao上标记@Mappee注解来将Dao加入到容器中（不推荐使用@Repository，在插入时有时候会出现问题）

* mapper.xml的namespace应该与Dao对应

  ```xml
  <mapper namespace="com.sise.cloud.dao.PaymentDao">
  ```

* mybatis的配置：

  ```yml
  mybatis:
    # mapper.xml存放位置
    mapper-locations: classpath:mapper/*.xml
    # 实体类不用写全限定类名
    type-aliases-package: com.sise.cloud.model
  ```

* mybatis插入时返回主键的写法：

  ```xml
  <insert id="create" useGeneratedKeys="true" keyProperty="id" parameterType="Payment">
        insert into payment(serial) values( #{serial} )
  </insert>
  ```

  **返回的主键在对应实体的keyProperty中，而不是方法的返回值。方法的返回值是影响行数。**

  ```java
  @PostMapping("/create")
  public CommonResult<Long> create(@RequestBody  Payment payment){
      Integer row = paymentService.create(payment);
      return row != null ? new CommonResult<>(200,"ok", payment.getId())
          : new CommonResult<>(500,"create error");
  }
  
  ```

* datasource配置：

  **注意serverTimezone的大小写。**

  ```yml
  spring:
    application:
      name: cloud-payment-service
    datasource:
      # 当前数据库操作类型
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/mcloud?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      username: root
      password: 123
  ```

* post方法传递json数据时，后端记得加上@RequestBody注解

#### devtool热部署

1. 添加maven依赖(子POM)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

2. 在pom文件中添加maven-plugin(父POM)

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3. enabling automatic build

![image-热部署idea设置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211204645960.png)

4. ctrl + shift + alt + /

![image-registry设置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211204742180.png)

#### Run Dashboard的配置

**在项目的存放路径下.idea目录下，有一个workspace.xml，找到RunDashboard的选项，添加配置**

```xml
<component name="RunDashboard">
    <option name="configurationTypes">
        <set>
            <option value="SpringBootApplicationConfigurationType" />
        </set>
    </option>
    <option name="ruleStates">
        <list>
            <RuleState>
                <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
            </RuleState>
            <RuleState>
                <option name="name" value="StatusDashboardGroupingRule" />
            </RuleState>
        </list>
    </option>
</component>
```

重启idea后，再次run项目就出现了。

#### maven出现KIX path building failed

idea maven设置添加参数

```java
-DarchetypeCatalog=internal -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true
```

![image-maven设置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211213212548.png)

#### 抽取出公用的api和实体类

* 新建一个moduel：cloud-api-common
* 将多个模块重复的代码抽取到这个模块
* maven clean instll ，将公用模块发布到仓库
* 其他module引入公用模块的坐标

#### 80端口被占用

在window上，80端口被占用的2种情况：

* SQL Server Reporting Services (MSSQLSERVER) ，SQL Server的日志系统
* IIS服务

解决方法（SQL Server Reporting Services ）：

1. 使用命令netstat -ano|findstr 80查找使用80端口的程序，可以看到是pid=4的的程序
2. 使用命令tasklist列出当前运行的所有线程，可以看到pid=4的程序竟然是system
3. 使用命名services.msc进入服务端口，找到SQL Server Reporting Services (MSSQLSERVER)，终止

解决方法（IIS）:

1. 使用管理员身份运行cmd
2. 使用net stop http停止http服务
3. sc config http start= disabled  **注意，=后面的空格不可少**



#### linux安装wget

1. linux查看版本

```shell
lsb_release -a
```

2. 根据linux版本下载wget命令

```
http://ftp.sjtu.edu.cn/centos/7/os/x86_64/Packages/
下载wget-1.14-18.el7_6.1.x86_64.rpm   
上传到阿里云服务器
```

3. 安装wget

```shell
rpm -vih wget-1.14-18.el7_6.1.x86_64.rpm
```

如果已经安装了wget，还提示wget command is not found？ 

```shell
# 1.执行以下命令查询wge命令的路径
whereis wge
# 系统返回以下信息。
wge：/usr/bin/wge
# 2.根据上述的路径，执行以下命令重命名即可。
cp /usr/bin/wge /usr/bin/wget
```



#### linux配置网卡

1. 使用ipconfig查看ip信息

2. vim /etc/sysconfig/network-scripts/ifcfg-eth0

```shell
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
ARPCHECK=no
IPADDR=172.16.47.149
NETMASK=255.255.240.0
GATEWAY=172.16.47.253
```



#### yum

```java
# 查询包安装的位置
rpm -ql 包名
# 查询所有已经安装的包
rpm  -qa  
# 查询包是否安装
rpm  -q
```





#### 修改阿里的yum源

1.  首先备份系统自带yum源配置文件/etc/yum.repos.d/CentOS-Base.repo

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2. 查看centos版本

```shell
lsb_release -a
```

3. 下载yum源配置文件到/etc/yum.repos.d

```shell
#CentOS7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

#CentOS6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

#CentOS5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

```

4. 运行yum makecache生成缓存

```shell
yum clean all
yum makecache
```



#### linux安装JDK

1. 下载rpm包，上传到linux服务器

2. rpm -ivh 安装包

3. rpm -qa | grep jdk 查找当前系统中安装的JDK包

4. rpm -ql jdk包名 | grep bin 查找JDK安装的目录

5. 配置环境变量vim /etc/profile

   ```shell
   export JAVA_HOME=/usr/java/jdk1.8.0_271-amd64
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   export PATH=$PATH:$JAVA_HOME/bin
   ```

6. 测试查看JDK版本  java  javac



#### linux安装maven

1. 打开maven的官网下载页，找到tar.gz压缩包，然后右键选择【复制链接地址】

2. 回到Linux服务器中，创建一个maven目录，使用`wget`命令将复制的链接进行下载。

3.  然后使用命令`tar -zxvf apache-maven-3.6.0-bin.tar.gz`，将压缩包进行解压，解压后我们会得到一个 `apache-maven-3.6.0`目录。

4. 我们cd进入到 `apache-maven-3.6.0`目录，然后执行命令`pwd`，显示maven的绝对路径。然后复制该路径

5. 执行命令`vim /etc/profile `编辑环境变量文件。

   ```shell
   export MAVEN_HOME=/root/app/maven/apache-maven-3.6.0
   ```

6. 将MAVEN_HOME追加到PATH环境变量中

   ```shell
   export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
   ```

7. 执行`source /etc/profile`命令来更新刚才的配置。

8. 执行命令`mvn -version`查看maven是否安装配置成功。



#### linux安装rabbitMQ

1. 安装erlang,添加源vim /etc/yum.repos.d/erlang-solutions.repo

```shell
[erlang-solutions]
name=CentOS $releasever - $basearch - Erlang Solutions
baseurl=https://packages.erlang-solutions.com/rpm/centos/$releasever/$basearch
gpgcheck=1
gpgkey=https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
enabled=1
```

执行命令：

```shell
rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
```

安装erlang：

```shell
yum install erlang -y
```

验证是否安装成功：

```shell
erl -version
```

![image-查看erlang版本](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201219203141782.png)

2. 安装rabbitMQ

下载：

```shell
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.14/rabbitmq-server-3.7.14-1.el7.noarch.rpm
```

安装：

```shell
yum install -y rabbitmq-server-3.7.13-1.el7.noarch.rpm
rpm -ivh rabbitmq-server-3.7.13-1.el7.noarch.rpm
```

安装过程中如果出现错误：

```shell
Failed dependencies: 	socat is needed by rabbitmq-server-3.7.14-1.el7.noarch
```

执行命令：

```shell
yum install socat
```

安装完成后在/usr/sbin目录下有4个关于rabbitMQ的命令：

![image-rabbitMQ命令](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201219222958331.png)

3. 启动rabbitMQ

> rabbitmq-server start，当然也可以使用 rabbitmqctl statr_app 来启动

4. 开启rabbitMQ界面管理

```shell
rabbitmq-plugins enable rabbitmq_management
```

5. 添加账号，授权远程访问

> 默认帐号guest只能用于本地访问，要远程访问可添加用户授权
>
> 添加新用户：rabbitmqctl add_user ming 123456
>
> 给新用户添加tags：rabbitmqctl set_user_tags ming administrator
>
> 授权：rabbitmqctl set_permissions ming ".*" ".*" ".*"
>
> 查看rabbitmqctl 命令的使用：rabbitmqctl 

6. 重启

> 关闭：rabbitmqctl stop
>
> 启动：rabbitmqctl start_app
>
> 浏览器输入 ip:15672 进入登录界面，输入刚才创建的用户名和密码即可进入

7. 开放端口，15672是web管理界面的端口，5672是MQ访问的端口

> 如果是阿里云的服务器，需要到安全组中开放5672和15672端口



#### linux安装nacos

1. 确保先安装了JDK和Maven

2. 上传tar.gz包到服务器，tar -zxvf xxx解压后，bin目录中运行

   ./startup.sh -m standalone



#### linux安装mysql

1. 检查是否已经安装过mysql

```yml
rpm -qa | grep mysql
```

2. 如果已经安装，执行删除命令

```shell
rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
```

3. 查询所有mysql对应的文件夹,删除相关目录或文件

```yml
whereis mysql
rm -rf /usr/bin/mysql /usr/include/mysql /data/mysql /data/mysql/mysql 
```

4. 检查mysql用户组和用户是否存在，如果没有，则创建

```shell
cat /etc/group | grep mysql
cat /etc/passwd |grep mysql
groupadd mysql
useradd -r -g mysql mysql
```

5. 下载mysql安装包

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

6. 解压到/usr/local/mysql目录下

```shell
tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

7. 在/usr/local/mysql目录下创建data目录

```shell
mkdir /usr/local/mysql/data
```

8. 更改mysql目录下所有的目录及文件夹所属的用户组和用户，以及权限

```shell
 chown -R mysql:mysql /usr/local/mysql
 chmod -R 755 /usr/local/mysql
```

9. 编译安装并初始化mysql,**务必记住初始化输出日志末尾的密码（数据库管理员临时密码）**

```shell
 cd /usr/local/mysql/bin
 ./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
```

10. 编辑配置文件my.cnf

```shell
vi /etc/my.cnf

[mysqld]
datadir=/usr/local/mysql/data
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=600
innodb_file_per_table=1
lower_case_table_names=1
character_set_server=utf8
```

11. 测试启动mysql服务器

```shell
/usr/local/mysql/support-files/mysql.server start
```

12. 添加软连接，并重启mysql服务

```shell
 ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
 ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
 service mysql restart
```

13. 登录mysql，修改密码(密码为步骤5生成的临时密码)

```shell
 mysql -u root -p
 set password for root@localhost = password('yourpass');
```

14. 开放远程连接

```shell
use mysql;
update user set user.Host='%' where user.User='root';
flush privileges;
```

15. 设置开机自动启动

```shell
1、将服务文件拷贝到init.d下，并重命名为mysql
[root@localhost /]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
2、赋予可执行权限
[root@localhost /]# chmod +x /etc/init.d/mysqld
3、添加服务
[root@localhost /]# chkconfig --add mysqld
4、显示服务列表
[root@localhost /]# chkconfig --list
```



### Eureka

#### 单机版Eureka服务端

1. 导入maven依赖

```xml
<dependencies>
    <!-- eureka-server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--监控-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!-- 一般通用配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

2. application.yml配置文件

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己就是注册中心,我的职责是维护服务实例,并不需要去检索服务
    fetch-registry: false
    # 查询服务和注册服务都需要依赖这个地址
    service-url:
      defalultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

3. 启动类标明是eureka服务端

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

4. 启动项目，访问服务器地址http://localhost:7001

![image-eureka服务端启动](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211222400955.png)



#### 单机版Eureka客户端

1. 导入Eureka client依赖

```xml
<!--eureka client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!--监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 修改application.yml配置文件，添加eureka client配置项

```yml
eureka:
  client:
    # 将自己注册进EurekaServer，默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认为true
    # 单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    # EurekaServer url
    service-url:
      defaultZone: http://localhost:7001/eureka/
```

3. 在启动类上添加注解@EnableEurekaClient

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class,args);
    }
}

```

4. 先启动EurekaServer，再启动EurekaClient

**必须先启动服务端，否则客户端连接会报错Connection refused: connect**

![image-eurekaServer](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211230437404.png)



#### 单机版和集群版的比较

**单机版本的Eureka会有单点故障的问题。**

![image-集群版Eureka](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201211234341772.png)

多台EurekaServer服务器之间的关系是怎样的？**互相注册，相互守望**



#### 集群版Eureka服务器

**所谓的集群版是指：有多台EurekaServer服务器，它们互相 注册，相互守望 。**

创建的步骤和单机版的Eureka服务器很类似，由于它们要互相注册，相互守望，一个hostname满足不了。

所有，我们需要修改host文件将127.0.0.1本地映射成2个hostname。

1. 修改host文件，C:\Windows\System32\drivers\etc

```java
# mcloud
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
```

##### 修改hosts不生效解决方法

* 直接修改hosts文件，一般情况下是保存失败的。需要保存到其他目录，在复制进C:\Windows\System32\drivers\etc目录，选择继续替换掉原来的host

* 如果还不生效，尝试刷新一下DNS缓存。使用命令ipconfig /flushdns

  

2. 修改application.yml配置文件

 7001端口的EurekaServer：

注意hostname 和 service-url的配置：

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己就是注册中心,我的职责是维护服务实例,并不需要去检索服务
    fetch-registry: false
    # 查询服务和注册服务都需要依赖这个地址
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka
```

7002端口的EurekaServer：

```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

3. 启动服务

![image-port7001](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212110342536.png)

![image-port7002](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212110414933.png)



#### 集群版Eureka客户端

集群版的客户端主要是application.yml配置文件不同：**service-url注册两个Eureka服务器的地址。**

order:

```yml
eureka:
  client:
    # 将自己注册进EurekaServer，默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认为true
    # 单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    # EurekaServer url
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

payment:

```yml
eureka:
  client:
    # 将自己注册进EurekaServer，默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认为true
    # 单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    # EurekaServer url
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```



#### 集群版服务提供者

由于有多台服务器提供服务，它们都要向注册中心注册服务。

只是application.yml的配置文件不同：比如server.port不同

**但是spring.application.name是相同的。集群版服务提供者暴露出一个相同的服务名称，但内部可能由多个端口提供服务。**

```yml
server:
  port: 8002

spring:
  application:
    name: cloud-payment-service
  datasource:
    # 当前数据库操作类型
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mcloud?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123


mybatis:
  # mapper.xml存放位置
  mapper-locations: classpath:mapper/*.xml
  # 实体类不用写全限定类名
  type-aliases-package: com.sise.cloud.model

eureka:
  client:
    # 将自己注册进EurekaServer，默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认为true
    # 单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    # EurekaServer url
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

```

那么，服务使用者order80该怎么去访问集群版的服务提供者提供的服务呢？

**只需要使用服务提供者暴露出来的服务名称。**

如下，统一使用CLOUD-PAYMENT-SERVICE访问8001和8002提供的服务。

经过测试，通过访问服务提供者application.yml中配置的小写的cloud-payment-service也是可以的。

```java
@RestController
@RequestMapping("/customer")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    private String ROOT = "http://CLOUD-PAYMENT-SERVICE";

    @GetMapping("/create")
    public CommonResult<String> create(Payment payment) {
        CommonResult commonResult = restTemplate.postForEntity(ROOT + "/payment/create", payment, CommonResult.class).getBody();
        return commonResult;
    }

    @GetMapping("/find/{id}")
    public CommonResult<Payment> findById(@PathVariable("id") Long id) {
        CommonResult commonResult = restTemplate.getForEntity(ROOT + "/payment/find/" + id, CommonResult.class).getBody();
        return commonResult;
    }

}
```

此时，如果使用order80去访问服务，会出现如下错误：

![image-集群服务提供者访问出现错误](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212131113323.png)

这是由于有8001和8002多个服务提供者，而order80不知道应该去访问哪个服务。

因为order80是通过restTemplate去访问的服务，应该配置restTemplate的负载均衡。

```java
@Configuration
public class BeanConfiguration {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

此时再使用order80去访问，就可以访问到服务了。

使用默认的负载均衡策略- 轮询。

![image-8001端口提供的服务](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212131502623.png)

刷新一下，可能看到的：

![image-8002端口提供的服务](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212131557705.png)



#### 微服务主机信息和ip信息的配置

1. 配置前需要导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 修改application.yml配置文件，设置instance-id 和 prefer-ip-address

```yml
eureka:
  client:
    # 将自己注册进EurekaServer，默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认为true
    # 单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    # EurekaServer url
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    # 主机名称
    instance-id: payment8002
    # 访问路径可以显示ip
    prefer-ip-address: true
```

![image-配置主机信息和ip信息](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212134024996.png)



#### 服务发现

对于注册进Eureka的服务，可以通过服务发现获取服务的信息。

1. 注入org.springframework.cloud.client.discovery.DiscoveryClient类

```java
@Autowired
private DiscoveryClient discoveryClient;

@Value("${spring.application.name}")
private String applicationName;

@GetMapping("/discovery")
public Map<String, Object> discovery() {
    Map map = new HashMap<>(8);

    // 获取EurekaServer上的所有服务名称
    List<String> services = discoveryClient.getServices();

    // 获取某个服务名称对应的所有实例信息
    List<ServiceInstance> instances = discoveryClient.getInstances(applicationName);

    map.put("services",services);
    map.put("instances",instances);

    return map;
}
```

2. 启动类上添加注解@EnableDiscoveryClient，==新版的springboot已经自动配置，可以不用加这个注解==

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class Payment8001Application {
    public static void main(String[] args) {
        SpringApplication.run(Payment8001Application.class,args);
    }
}
```

输出结果：

![image-服务发现获取服务信息](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212140312487.png)



#### 自我保护

1. 为什么需要自我保护的机制？

有时候，EurekaClient可以正常运行，但是与EurekaServer网络不通的情况，自我保护机制不会立刻将EurekaClient从服务注册表中删除。

2. 什么是自我保护机制？

EurekaServer将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。

当某时刻某一个微服务不可用了（在一定时间内（默认90s）没有接收到某个微服务实例的心跳），EurekaServer不会立即清理，依旧会对该微服务的信息进行保存。

3. 如何关闭自我保护机制？

EurekaServer的自我保护机制默认是开启的。可以在服务端的配置文件中设置：

```yml
eureka:
	server:
        # 设置接收心跳的时间间隔
        eviction-interval-timer-in-ms: 2000
        # 关闭自我保护,默认值为true，保证不可用服务及时剔除	
        enable-self-preservation: false
```

为了方便测试，设置EurekaClient发送心跳的时间间隔和服务器等待时间上限：

如下，每隔1s，客户端向服务器发送心跳。服务端在接收到最后一次心跳后，最长等待2s。

如果2s后没有再次接收到客户端的心跳信息，服务端又关闭了自我保护机制的话，客户端实例会被服务端从服务注册表中删除。

```yml
eureka:
  instance:
    # 主机名称
    instance-id: payment8001
    # 访问路径可以显示ip
    prefer-ip-address: true
    # 客户端向服务端发送心跳的时间间隔，默认为30s
    lease-renewal-interval-in-seconds: 1
    # 服务端在收到最后一次心跳后等待时间上限,默认为90s
    lease-expiration-duration-in-seconds: 2
```

![image-自我保护机制](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201212153620657.png)



### Ribbon

引入spring-cloud-starter-netflix-eureka-client的时候就自动引入了ribbon



#### 修改默认的负载均衡算法

1. 实现自己的配置类，返回新的IRule实例。

   **注意，配置类不能放在启动类能扫描到的包或子包下。**

![image-配置类的存放路径](C:\Users\zjm16\AppData\Roaming\Typora\typora-user-images\image-20201213163211853.png)

```java
@Configuration
public class CustomerRuleConfiguration {

    @Bean
    public IRule iRule() {
        // 修改为随机的负载均衡算法
        return new RandomRule();
    }
}

```

2. 在启动类上加上注解RibbonClient，指定配置类

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "cloud-payment-service",configuration = CustomerRuleConfiguration.class)
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

#### Ribbon负载均衡算法

**rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务位置下标**

每次服务重启后，rest接口计数从1开始。



使用自旋锁和CAS实现轮询算法：

1. 定义接口，定义方法

```java
public interface LoadBalance {
    /**
     * 从多个服务实例中选择一个实例
     * @param instances 多个服务实例
     * @return ServiceInstance
     */
    ServiceInstance instance(List<ServiceInstance> instances);
}
```

2. 实现LoadBanlance接口

```java
@Component
public class RoundLoadBalance implements LoadBalance{

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    @Override
    public ServiceInstance instance(List<ServiceInstance> instances) {
        int index = this.getIndexAndIncrement(instances.size());
        return instances.get(index);
    }

    /**
     * 获取当前服务的下标,并自增1
     * @param size 服务的集群实例总数量
     * @return 实际调用服务位置下标
     */
    private int getIndexAndIncrement(int size) {
        // 自旋锁 和 CAS 实现轮询算法
        for (;;){
            int current = atomicInteger.get();
            int next = (current + 1) % size;
            // 如果下标更新成功了,返回。更新不成功,自旋直至更新成功
            if(atomicInteger.compareAndSet(current, next)) {
                return next;
            }
        }
    }
}
```

3. 注入LoadBalance，调用服务

在调用自己实现的LoadBalance之前，记得去掉RestTemplate的@LocdBalance注解。

```java

@Autowired
private LoadBalance loadBalance;

@Autowired
private DiscoveryClient discoveryClient;

@GetMapping("round/find/{id}")
public CommonResult findByIdAndRound(@PathVariable("id") Long id) {
    List<ServiceInstance> instances = discoveryClient.getInstances("cloud-payment-service");
    ServiceInstance instance = loadBalance.instance(instances);
    URI uri = instance.getUri();
    return restTemplate.getForObject(uri +"/payment/find/" + id, CommonResult.class);
}

```



### OpenFeign

#### 基本使用

OpenFeign集成了Ribbon和RestTemplate，提供了一种接口+@FeginClient进行服务调用的机制。

1. 引入maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. application.yml配置文件

此处不将其注册为EurekaService的服务，设置了register-with-eureka: false

```yml
server:
  port: 80

spring:
  application:
    name: cloud-openFeign-order-service

eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

```

3. 编写启动类，加上@EnableFeignClients

```java
@SpringBootApplication
@EnableFeignClients
public class OrderOpenFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderOpenFeignApplication.class, args);
    }
}
```

4. 编写接口，加上@FeignClient

@FeignClient的name属性是服务提供者的服务名称。

接口中的方法与服务提供者Controller的方法相同，url也应该相同。

服务消费者的PaymentService:

```java
@FeignClient(name = "cloud-payment-service")
public interface PaymentService {

    @PostMapping("/payment/create")
    CommonResult<Long> create(@RequestBody Payment payment);

    @GetMapping("/payment/find/{id}")
    CommonResult<Payment> findById(@PathVariable("id") Long id);
}
```

服务提供者的PaymentController:

```java
@RestController
@RequestMapping("/payment")
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @Value("${server.port}")
    private String port;

    @PostMapping("/create")
    public CommonResult<Long> create(@RequestBody  Payment payment){
        Integer row = paymentService.create(payment);
        return row != null ? new CommonResult<>(200,"ok" + port, payment.getId())
                : new CommonResult<>(500,"create error");
    }

    @GetMapping("/find/{id}")
    public CommonResult<Payment> findById(@PathVariable("id") Long id) {
        Payment payment = paymentService.findById(id);
        return payment != null ? new CommonResult<>(200,"ok" + port,payment)
                : new CommonResult<>(404,"not found");
    }
}
```

4. 编写服务消费者的controller，直接调用PaymentService,不需要使用Ribbon

```java
@RestController
@RequestMapping("/customer")
public class OrderController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/create")
    public CommonResult<Long> create(Payment payment) {
        return paymentService.create(payment);
    }

    @GetMapping("/find/{id}")
    public CommonResult<Payment> findById(@PathVariable("id") Long id) {
       return paymentService.findById(id);
    }
}
```

 

#### 超时机制

OpenFeign的默认超时时间是1s，请求超出这个时间，会报timeout的错误。

设置超时时间：

由于OpenFeign底层是ribbon，设置的是ribbon的属性。

```yml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```



#### 日志记录

OpenFeign提供了日志对接口的调用情况进行监控和输出。

日志级别：

* NONE：默认的，不显示任何日志
* BASIC：仅记录请求方法、URL、响应状态码以及执行时间
* HEADERS：除了BASIC定义的信息之外，还有请求和响应的头信息
* FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文以及元数据

1. 添加配置类，设置日志级别

```java
@Configuration
public class FeginConfiguration {

    @Bean
    public Logger.Level logLevel() {
        return Logger.Level.FULL;
    }
}
```

2. application.yml配置文件

```yml
logging:
  level:
    # 设置类的日志级别
    com.sise.cloud.service.PaymentService:  debug
```



### Hystrix

#### 基本概念

> 服务降级（fallback）

哪些情况会发生服务降级：

* 异常
* 超时
* 服务熔断
* 线程池/信号量打满



> 服务熔断（break）

类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级



> 服务限流（flow limit）

秒杀高并发等操作，一秒钟N个，有序进行。

服务降级 -> 进而熔断 -> 限流



#### 服务降级



1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

2.  @HystrixCommand注解的使用

@HystrixCommand指定当发生超时或异常时，执行fallbackMethod指定的降级方法。

@HystrixProperty指定超时上限，当超过这个时间时，会执行fallbackMethod指定的降级方法。

```java
 @HystrixCommand(fallbackMethod = "timeoutOrExceptionHandler",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "1000")
    })
    public String timeout(String id) {
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池：" + Thread.currentThread().getName() + "hystrix,timeout,id:" + id + "耗时:3s";

    }

    public String timeoutOrExceptionHandler(String id) {
        return "发生超时，或者发生异常. id:" + id;
    }
```

3. 启动类开启Hystrix

```java
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix
public class HystrixPayment8001Application {
    public static void main(String[] args) {
        SpringApplication.run(HystrixPayment8001Application.class, args);
    }
}
```

细节：

1. fallbackMethod指定的方法可以放在controller层，但是指定的方法和@HystrixCommand必须在同一个文件，否则报错fallback method wasn't found
2. 若发生异常，异常并不会抛出，直接执行了fallbackMethod指定的方法
3. 服务降级可以添加在服务器，也可以添加在客户端
4. 若客户端和服务端都做了服务降级，优先执行客户端的降级方法
5. fallbackMethod方法的参数要和对应的服务的方法形参类型一致



#### 全局服务降级

为每一个方法写一个fallbackMethod真的是太麻烦了。我们希望有一个全局的方法，当我方法上没指定fallbackMethod的时候，使用全局的fallbackMethod方法，当方法上指定了fallbackMethod的时候，使用方法上指定的fallbackMethod方法。

1. 使用@DefaultProperties在类上标记fallbackMethod
2. 使用@HystrixCommand在方法上标记，当方法出现异常或超时，又没有指定fallbackMethod时，使用@DefaultProperties标记的fallbackMethod方法

```java
@RestController
@RequestMapping("/customer")
@DefaultProperties(defaultFallback = "timeoutOrExceptionHandler")
public class OrderController {

    @Autowired
    private PaymentService paymentService;

    @RequestMapping("/ok/{id}")
    @HystrixCommand
    String ok(@PathVariable("id") String id) {
        int a = 10 / 0;
        return paymentService.ok(id);
    }

    //@HystrixCommand(fallbackMethod = "timeoutOrExceptionHandler",commandProperties = {
    //        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "6000")
    //})
    @RequestMapping("/timeout/{id}")
    @HystrixCommand
    String timeout(@PathVariable("id") String id) {
        return paymentService.timeout(id);
    }

    String timeoutOrExceptionHandler() {
        return "80发生超时，或者发生异常.";
    }
}

```

全局服务降级可以为同一个类中的方法统一一个fallbackMethod，但是与类形成了耦合，无法在多个类中共用。



#### Feign支持的松耦合服务降级

当使用Fegin的时候，调用的方法都是来自一个标记了@FeignClient的接口，我们就可以利用这个接口结合Feign，对接口中的每一个方法实现一个fallbackMethod方法。

1. 编写一个类实现提供服务的接口，并加入到容器当中

当在服务接口指定了fallbackMethod对应的类，而这个类没有加入到容器中时，启动的时候扫描不到，会报错

```java
@Component
public class PaymentFallbackMethod implements PaymentService {
    @Override
    public String ok(String id) {
        return "paymentService ok method is exception or timeout";
    }

    @Override
    public String timeout(String id) {
        return "paymentService timeout method is exception or timeout";
    }
}
```

2. 在服务提供接口指定fallbackMethod

```java
@FeignClient(name = "cloud-hystrix-payment-service",fallback = PaymentFallbackMethod.class)
public interface PaymentService {

    @RequestMapping("/payment/ok/{id}")
    String ok(@PathVariable("id") String id);


    @RequestMapping("/payment/timeout/{id}")
    String timeout(@PathVariable("id") String id);
    
}
```

3. application.yml开启Feign对Hystrix的支持

```yml
feign:
  hystrix:
    enabled: true
```



#### 服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制。当某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路。**

在SpringCloud框架里，熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，默认是5s内20次调用失败，就会启动熔断机制。

熔断机制的注解是@HystrixCommand.



下列代码表明：在10s内服务调用超过了10次，且失败了6次（60%），会启动熔断机制

```java
 @HystrixCommand(fallbackMethod = "timeoutOrExceptionHandler",commandProperties = {
            // 开启服务熔断
            @HystrixProperty(name="circuitBreaker.enabled",value = "true"),
            // 请求次数，次数超过阈值后，才会开启断路器
            @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value = "10"),
            // 时间范围
            @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
            // 失败率达到多少后跳闸
            @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value = "60"),
    })
    public String test(int id) {
        if(id < 0) {
            throw new RuntimeException("id 不能为负数");
        }
        return "ok,id:" + UUID.randomUUID();
    }

    public String timeoutOrExceptionHandler(int id) {
        return "8001发生超时，或者发生异常. id:" + id ;
    }
```

三个重要参数：

* 快照时间戳：断路器统计一些请求和错误信息的统计时间范围，默认为最近的10秒
* 请求总数阈值：在快照时间窗内，必须满足请求总数阈值才有资格熔断。默认值是20，意味着在10秒内，如果调用次数不足20次，即使所有的请求都超时或其他原因失败，断路器都不会打开
* 错误百分比阈值：当请求总数在快照时间窗内超过了阈值，比如，发生了30次调用，而有15次调用失败，也就是50%的错误百分比，这时候断路器就会打开。默认值是50%



**断路器开启/关闭的条件：**

* 当满足一定请求阈值的时候（默认10秒内超过20个请求）
* 并且失败率达到阈值的时候（默认10秒内超过50%的请求失败）
* 到达以上阈值，断路器将会开启，所有的请求都不会进行转发
* 一段时间之后（默认是5秒），这个时间断路器是半开状态，会让其中一个请求进行转发，如果成功，断路器会关闭，若失败，继续开启，重复以上步骤

```java
断路器的打开和关闭,是按照一下5步决定的
  	1,并发此时是否达到我们指定的阈值
  	2,错误百分比,比如我们配置了60%,那么如果并发请求中,10次有6次是失败的,就开启断路器
  	3,上面的条件符合,断路器改变状态为open(开启)
  	4,这个服务的断路器开启,所有请求无法访问
  	5,在我们的时间窗口期,期间,尝试让一些请求通过(半开状态),如果请求还是失败,证明断路器还是开启状态,服务没有恢复
  		如果请求成功了,证明服务已经恢复,断路器状态变为close关闭状态
```





### Gateway

#### 服务搭建

1. 引入依赖，注意gateway不需要引入web和actuator

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

2. application.yml配置路由

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway-service
  cloud:
    gateway:
      routes:
        - id: payment_rout
          # 真实的访问url
          uri: http://localhost:8001
          # **表示匹配参数
          predicates:
            - Path=/payment/find/**

        - id: payment_route
          uri: http://localhost:8001
          predicates:
            - Path=/payment/create/**
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

3. 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class Gateway9527Application {
    public static void main(String[] args) {
        SpringApplication.run(Gateway9527Application.class, args);
    }
}
```

4. 测试

```java
http://localhost:9527/payment/find/1
http://localhost:8001/payment/find/1
访问的是相同的页面，隐藏了8001端口，对外暴露出9527的端口
```



#### 编码方式配置路由

```java
@Configuration
public class GatewayConfiguration {

    /**
     * 注入RouteLocatorBuilder构建RouteLocator
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customerRouter(RouteLocatorBuilder builder) {
        System.out.println("fuck");
        RouteLocatorBuilder.Builder routes = builder.routes();
        // 构建一个/blog 指向 http://www.caixukun8.top
        routes.route("blog_route", r -> r.path("/fuck").uri("http://news.baidu.com/guonei"));
        return routes.build();
    }
}
```



#### 通过微服务名实现动态路由

上面的uri配置是写死在application.yml文件中的，我们可以利用gateway的动态路由，通过微服务的名称，负载均衡的查找服务。

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway-service
  cloud:
    gateway:
      # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_rout
          # 真实的访问url
          #uri: http://localhost:8001
          # lb是写死的,loadBalance,表示负载均衡
          uri: lb://cloud-payment-service
          # **表示匹配参数
          predicates:
            - Path=/payment/find/**

        - id: payment_route
          #uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/create/**

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

当访问http://localhost:9527/payment/find/1的时候，通过负载均衡去查找8001或8002端口提供的服务。



### Config

***

#### 读取远程仓库的配置文件

1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

2. application.yml配置文件

* 配置中心无需配置name属性和profile属性，只有别的服务来访问配置中心时，别的服务才需要配置name属性和profile属性

```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-service
  cloud:
    config:
      # 分支
      label: master
      # 文件名-前面的
      # name: config
      # 文件名-后面的
      # profile: dev
      # 远程仓库地址
      server:
        git:
          uri: git@github.com:JinMing8/mcloud.git
          # 配置文件在远程仓库的存放目录
          search-paths:
            - mcloud
```

3. 启动类

```java
@SpringBootApplication
@EnableConfigServer
public class Config3344Application {
    public static void main(String[] args) {
        SpringApplication.run(Config3344Application.class, args);
    }
}

```

4. 测试

假设github上存在以下两个配置文件：

![image-20201219142706651](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201219142706651.png)

通过浏览器访问：[http://localhost:3344/master/config-prod.yml]()得到以下结果：

```yml
config:
  info: mater branch,config-prod,version 1
```

或者通过[http://localhost:3344/config-prod/master]() 得到JSON格式的返回结果：

```json
{
  "name": "config-prod",
  "profiles": [
    "master"
  ],
  "label": null,
  "version": "6abe5f2cbb64765bc7ffa0ca3a8574f8b57256f8",
  "state": null,
  "propertySources": [
    {
      "name": "git@github.com:JinMing8/mcloud.git/config-prod.yml",
      "source": {
        "config.info": "mater branch,config-prod,version 1"
      }
    }
  ]
}
```

如果访问的路径在远程仓库中不存在，会返回以下的结果：

```json
{
  "name": "config-prod.yml",
  "profiles": [
    "master"
  ],
  "label": null,
  "version": "6abe5f2cbb64765bc7ffa0ca3a8574f8b57256f8",
  "state": null,
  "propertySources": [
    
  ]
}
```

如果访问的文件在远程仓库中不存在，会返回以下的结果：

```json
{}
```



#### 配置中心的搭建

以上只是单独创建了一个服务，去访问远程仓库的配置文件。

如果想让它成为配置中心，也非常简单，将其注册成为EurekaSerrver的一个服务即可。

```yml
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

启动类上加入注解：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class Config3344Application {
    public static void main(String[] args) {
        SpringApplication.run(Config3344Application.class, args);
    }
}
```



#### 其他服务访问配置中心

其他服务不直接访问访问远程仓库获取配置信息，而是访问配置中心获取配置文件的信息。

1. 引入依赖，config的客户端

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2. bootstarp.yml的配置文件

* config的客户端使用的配置文件是bootstrap.yml，而不是application.yml

> application.yml 和 bootstrap.yml文件的区别：
>
> |      application.yml       | bootstrap.yml                                                |
> | :------------------------: | ------------------------------------------------------------ |
> |    用户级别的资源配置项    | 系统级别，优先级更加高                                       |
> | 上下文是ApplicationContext | 上下文是BootstrapContext,是ApplicationContext的父上下文。BootstrapContext负责从外部源加载配置属性并解析配置，这两个上下文共享一个从外部获取的Environment |
>
> * Bootstrap有高优先级，默认情况下，它们不会被本地配置覆盖。
> * Bootstrap和Application有不同的上下文，保证配置的分离

```yml
server:
  port: 3355

spring:
  application:
    name: cloud-config-clinet-service
  cloud:
    config:
      # 分支
      label: master
      # 配置文件-前面的
      name: config
      # 配置文件-后面的
      profile: dev
      # 读取配置文件的地址
      uri: http://localhost:3344

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

3. 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class Config3355Application {

    public static void main(String[] args) {
        SpringApplication.run(Config3355Application.class, args);
    }
}

```

4. 编写测试Controller

```java
@RestController
public class ConfigController {
	// 直接读取配置文件的信息
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String configInfo() {
        return configInfo;
    }
}
```

上面的controller中直接从配置文件中读取配置信息，如果配置中心中不存在对应的配置中心，@Value读取失败，启动的时候就报错了。



5. 测试

访问[http://localhost:3355/configInfo]()，返回以下结果：

```json
mater branch,config-prod,version 1
```

如果访问[http://localhost:3355/master/config-prod.yml](),会报404



此时，我们修改远程仓库的配置文件，将version 1 改成 version 2:

```yml
config:
    info: mater branch,config-prod,version 2
```

再次访问[http://localhost:3355/configInfo]()，结果并没有刷新：

```json
mater branch,config-prod,version 1
```

如果访问[http://localhost:3344/master/config-prod.yml]，返回结果是修改后的配置文件：

```json
config:
  info: mater branch,config-prod,version 2
```

**也就是说，修改了远程仓库的配置文件，配置中心会立即刷新，而其他服务去访问配置中心，却不会刷新。**



#### 手动刷新其他服务的配置

1. 引入actuator监控依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 配置文件中暴露监控端口

```yml
# 暴露监控端口
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

3. 在需要刷新配置信息的类上添加注解@RefreshScope

```java
@RestController
@RefreshScope
public class ConfigController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String configInfo() {
        return configInfo;
    }
}
```

4. 利用actuator监控的特点，发生POST请求刷新服务

利用curl命令，发生POST请求到[http://localhost:3355/actuator/refresh].刷新3355这个端口的配置信息

```shell
curl -X POST http://localhost:3355/actuator/refresh
```



通过@RefreshScope和发生post请求去刷新，这种方式虽然可以刷新配置信息。但是这种方式不是最好的，如果有100个服务，需要手动发送100次POST请求(或许你可以使用脚本)。

由于100个服务都是从配置中心读取的配置文件，那么我们可不可以只刷新配置中心，让配置中心去刷新其他服务的配置信息呢？这就需要利用Spring Cloud Bus 。



### Spring Cloud Bus

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能。

Spring Cloud Bus目前支持RabbitMQ和Kafka。



#### 基本原理

每个ConfigClient实例都监听MQ中同一个topic（默认是SpringCloudBus）,当一个服务刷新数据的时候，它会把这个信息放入到topic中，这样其他监听同一topic的服务就能得到通知，然后去更新自身的配置信息。



那么，我们就有2种方案来实现配置信息的刷新：

* 通知ConfigClient中的其中一个客户端，这个客户端刷新数据的时候，会把更新的信息放入到topic中，其他客户端得到通知而更新自身配置
* 通知配置中心，由配置中心广播给其他客户端

我们采用通知配置中心的方案来更新配置信息，第一种方案有如下缺点：

* 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责
* 破坏了微服务各个节点的对等性



#### 通知配置中心，实现配置的自动刷新

1. 配置中心引入actuator,rabbitmq依赖

```xml
<!-- 添加消息总线RabbitMQ支持 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
<!--监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 配置文件中设置rabbitMQ设置,暴露监听端口

配置中心完整的配置文件如下：

```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-service
  cloud:
    config:
      # 分支
      label: master
      # 远程仓库地址
      server:
        git:
          uri: git@github.com:JinMing8/mcloud.git
          # 配置文件在远程仓库的存放目录
          search-paths:
            - mcloud
  # rabbitmq配置
  rabbitmq:
    host: 47.96.224.198
    port: 5672
    username: ming
    password: 123456

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
# 暴露监听端口
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```

3. ConfigClient也需要引入rabbitMQ依赖，并在配置文件中引入rabbitMQ的支持

ConfigClient完整的配置文件如下：

```yml
server:
  port: 3355

spring:
  application:
    name: cloud-config-clinet-service
  cloud:
    config:
      # 分支
      label: master
      # 配置文件-前面的
      name: config
      # 配置文件-后面的
      profile: prod
      # 读取配置文件的地址
      uri: http://localhost:3344
  #rabbitMQ配置
  rabbitmq:
    host: 47.96.224.198
    port: 5672
    username: ming
    password: 123456


eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

4. 测试

在远程仓库中修改配置文件信息后，发送POST请求通知配置中心，其他ConfigClient会收到配置中心的通知，从而更新自身的配置信息。

```shell
curl -X POST http://localhost:3344/actuator/bus-refresh
```



#### 定点通知

所谓定点通知，即通知部分ConfigClient，而不是全部的ConfigClient。

```java
curl -X POST http://localhost:${serverport}/actuator/bus-refresh/${destnation}
```

/bus/refresh请求不再发送到具体的服务实例上，而是发给配置中心，并通过destnation参数指定需要更新配置的服务实例。

这里的destnation实际上是服务名称（spring.application.name指定）+端口号。



比如上面的栗子，只通知localhost:3355这个服务实例更新配置：

```shell
curl -X POST http://localhost:3344/actuator/bus-refresh/cloud-config-clinet-service:3355
```



### Spring Cloud Stream

#### 通信模型

![image-Stream通信模型](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201219235538034.png)

> Source和Sink

* 发送消息端（生产者）是Source
* 消息接收端（消费者）是Sink

> Channel

队列的一种抽象，在消息通讯系统中就是实现存储和转发的媒介。



#### 构建生产者

1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

2. 配置文件

生产者的配置文件：与stream整合需要一个binder，而binder指定消息服务的具体环境（比如使用rabbitMQ的host，username等）

**rabbitmq的配置是写在spring内的。**

```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provide-service
  cloud:
    stream:
      binders:
        # 自定义一个binder的名称，用于stream的整合配置
        default-rabbit:
          # 消息组件类型
          type: rabbit
          # rabbitMQ相关的环境配置
          environment:
            spring:
              rabbitmq:
                virtual-host: /
                host: 47.96.224.198
                port: 5672
                username: ming
                password: 123456
      # stream的整合配置
      bindings:
        output:
          # 生产者生产的消息放到哪个通道
          destination: studyExchange
          # 消息类型，本次为json，如果是文本则设置为text/plain
          content-type: text/plain
          # 设置要绑定的消息服务的具体设置
          binder: default-rabbit


eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    # 主机名称
    instance-id: send-8801
    # 显示ip
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5



```

3. 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamProvide8801Application {
    public static void main(String[] args) {
        SpringApplication.run(StreamProvide8801Application.class, args);
    }
}
```

4. Source类生产消息

```java
/**
 *@Author Ming
 *@Date 2020/12/20 00:20
 *@Description 消息生产者
 * 1. Source 是org.springframework.cloud.stream.messaging.Source包的
 * 2. 不需要使用@Service将其加入到容器，@EnableBinding已经指定这是一个Binder,
 *      将Channel和exchange绑定在一起
 * 3. 注意不要引错包
 * import org.springframework.messaging.Message;
 * import org.springframework.messaging.MessageChannel;
 * import org.springframework.messaging.support.MessageBuilder;


 */

@EnableBinding(Source.class)
public class MessageProvider {

    /**
     * 注入消息管道,messageChannel接口有多个实现类，我们需要使用的是Source类,
     */
    @Autowired
    @Qualifier("output")
    private MessageChannel output;

    public String send() {
        // 构建Message,消息内容为UUID
        Message<String> message = MessageBuilder.withPayload(UUID.randomUUID().toString()).build();
        // 输出消息内容
        System.out.println(message.getPayload());
        // 将消息发送至Channel
        output.send(message);
        return message.getPayload();
    }
}

```

```
MessageChannel有多个实现类，这里的output与配置文件中配置的需要一致。
```

![image-MessageChannel的注入](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201220003426084.png)

5. 测试

登录rabbitMQ的管理界面，可以看到我们自定义的studyExchange:

![image-studyExchange](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201220153356198.png)



#### 构建消费者

1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

2. 配置文件

消费者主要是配置rabbitMQ的相关配置，并且配置从哪个Channel取出消息

```yml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-customer-service
  cloud:
    stream:
      binders:
        # 自定义一个binder的名称，用于stream的整合配置
        default-rabbit:
          # 消息组件类型
          type: rabbit
          # rabbitMQ相关的环境配置
          environment:
            spring:
              rabbitmq:
                virtual-host: /
                host: 47.96.224.198
                port: 5672
                username: ming
                password: 123456
      # stream的整合配置
      bindings:
        input:
          # 消费者消费的消息从哪个通道取
          destination: studyExchange
          # 消息类型，本次为json，如果是文本则设置为text/plain
          content-type: text/plain
          # 设置要绑定的消息服务的具体设置
          binder: default-rabbit


eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    # 主机名称
    instance-id: customer-8802
    # 显示ip
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5


```

3. 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamCustomer8802Application {
    public static void main(String[] args) {
        SpringApplication.run(StreamCustomer8802Application.class, args);
    }
}
```

4. 业务类取出消息

方法上使用注解@StreamListener，方法形参绑定发送者发送的消息类型。

```java
@RestController
@EnableBinding(Sink.class)
public class CustomerController {

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void get(Message<String> message) {
        String output = "消费者:" + serverPort +",获得的消息是："+message.getPayload();
        System.out.println(output);
    }
}

```

5. 测试

访问生产者[http://localhost:8801/provide/send]()，产生一条消息:

```json
a9841d4e-5510-4395-ada5-53edad88b5f7
```

此时消费者8802接收到消息：

```java
消费者:8802,获得的消息是：a9841d4e-5510-4395-ada5-53edad88b5f7
```

如果此时还有其他的消费者，比如消费者8803：

```java
消费者:8803,获得的消息是：a9841d4e-5510-4395-ada5-53edad88b5f7
```

可见，生产者生产的一条消息，多个消费者都收到了，并且重复消费了。



#### 分组解决重复消费问题

在Stream中，处于同一个group中的多个消费者是竞争关系，能够保证消息只会被其中一个消费者消费一次。

不同group是可以重复消费的。

查看rabbitMQ的界面，可以查看，默认情况下两个消费者属于不同的分组，因此它们可以重复消费。

![image-group](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201220160328224.png)



我们可以为消费者设置相同的分组，解决重复消费的问题。分别在多个消费者的配置文件中配置group:

```yml
 # stream的整合配置
      bindings:
        input:
          # 消费者消费的消息从哪个通道取
          destination: studyExchange
          # 消息类型，本次为json，如果是文本则设置为text/plain
          content-type: text/plain
          # 设置要绑定的消息服务的具体设置
          binder: default-rabbit
          # 指定分组
          group: customerA
```



重启后，生产者生产一条消息，只会被同组内的一个消费者消费。

查看rabbitMQ管理界面也可以看到我们设置的分组生效了：

![image-同一个group](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201220160757195.png)



#### 持久化问题

假设消费者的服务挂掉了，但是生产者还一直在生产消息，那么这么消息会被怎么处理呢？

* 如果指定了分组，这些消息会被持久化，当消费者重新启动的时候，这些消息会被消费
* 如果没有指定分组，这些消息会丢失



测试：

消费者8802不指定分组，消费者8803指定分组customerA。同时关闭消费者8802和消费者8803，访问生产者生产3条消息。重新消费者服务:

消费者8803指定了分组，一启动消费之前持久化的消息：

```java
消费者:8803,获得的消息是：756d6ec3-23d3-4af4-93f3-5a650ca6ee67
消费者:8803,获得的消息是：f37512be-f8e4-42f4-9a59-b53ec1e5668e
消费者:8803,获得的消息是：35b57408-263c-4dd4-9aa5-b0b00a5e773e
```

而消费者8802重启之后，之前的消息丢失。



### Sleuth

***

在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的服务节点调用来协同产生最后的请求结果，每一个前端请求都回形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延迟或者错误都会引起整个请求最后的失败。而Sleuth就是用于追踪每个请求的整体链路。



#### 基本使用

1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

2. 修改配置文件

```yml
spring:
  application:
    name: cloud-order-service
  zipkin:
    # 指定数据显示在哪里
    base-url: http://localhost:9411
    sleuth:
      sampler:
        # 采样率,介于0到1直接,1则表示全部采集
        probability: 1
```

3. 启动java -jar java -jar zipkin-server-2.12.9-exec.jar

这个jar包需要单独下载，这个jar就是图形界面的展示。

启动完成后，访问[http://localhost:9411/zipkin/](),可以查看链路。

span指的是访问的url。

![image-zipkin](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201220164008117.png)

### Nacos

#### 服务提供者搭建

1. 导入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2. 配置文件

nacos不需要其他的服务注册模块了，不需要引入Eureka那一套了。

**注意server-addr的写法，写成[http://47.96.224.198:8848]()是不行的，启动会报错。**

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-service
  cloud:
    nacos:
      discovery:
        server-addr: 47.96.224.198:8848 # nacos服务器地址
```

3. 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosPayment9001Application {
    public static void main(String[] args) {
        SpringApplication.run(NacosPayment9001Application.class, args);
    }
}
```

4. 测试

登录[http://47.96.224.198:8848/nacos]，账户密码默认都为nacos。可以看到服务已经完成注册了。

![image-服务注册](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221194659396.png)

访问[http://localhost:9001/payment/nacos/1](),返回结果符合预期。



#### 服务消费者搭建

1. 导入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2. 配置文件

```yml
server:
  port: 83

spring:
  application:
    name: nacos-order-service
  cloud:
    nacos:
      discovery:
        server-addr: 47.96.224.198:8848


# 自定义配置，要调用的服务地址
service-url:
  payment-service-url: http://nacos-payment-service
```

3. 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosOrder83Application {
    public static void main(String[] args) {
        SpringApplication.run(NacosOrder83Application.class, args);
    }
}

```

4. 业务类

```java
@RestController
@RequestMapping("order")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${service-url.payment-service-url}")
    private String paymentService;

    @GetMapping("/nacos/{id}")
    public String getPayment(@PathVariable("id") String id) {
        return restTemplate.getForObject(paymentService+"/payment/nacos/"+id,String.class);
    }
}
```

5. 配置类

配置restTemplate，注意加上负载均衡的注解。

```java
@Configuration
public class ApplicationContextConfiguration {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

6. 测试

打开nacos的管理界面，消费者order83已经完成注册。

![image消费者完成注册](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221195103011.png)

访问[http://localhost:83/order/nacos/1](),返回结果预期。多次刷新页面，打印的端口也在变化，实现了负载均衡。



#### 配置中心

通过公式 **${prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}** 从nacos中拉取配置。

> prefix默认为spring.application.name指定的值，也可以通过spring.cloud.nacos.config.prefix配置
>
> spring.profiles.active为当前环境对应的profile,(dev,prod，test等)
>
> file-extension为配置内容的数据格式，可以通过spring.cloud.nacos.file-extension来配置

![image-配置文件命名规则](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221203317319.png)

1. 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2. 配置文件，bootstrap.yml 和 application.yml

bootstrap.yml:

除了指定服务注册中心的地址之外，还需要执行配置中心的地址。

```yml
server:
  port: 3377
spring:
  application:
    name: nacos-config-service
  cloud:
    nacos:
      discovery:
        server-addr: 47.96.224.198:8848
      config:
        # 配置中心的地址
        server-addr: 47.96.224.198:8848
        # 只支持yaml,不支持yml
        file-extension: yaml
        prefix: config
```

application.yml:

```yml
spring:
  profiles:
    active: dev
```

3. 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfig3377Application {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfig3377Application.class, args);
    }
}
```

4. 业务类

通过Spring Cloud的注解@RefreshScope实现配置的自动刷新，与Spring Clound Config相比，不再需要发送post请求。

```java
@RestController
@RefreshScope
public class ConfigController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

5. 测试

在nacos的配置中心根据${profix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}创建配置文件：config-dev.yaml

![image-config-dev.yaml](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221202509643.png)

启动服务nacos-config-3377，访问[http://localhost:3377/config/info](),得到以下结果：

```json
nacos config,DEFAULT_GROUP,version = 1
```

从nacos的配置中心获取到了文件，尝试修改config-dev.yaml文件，将其version = 2

```yml
config:
    info: nacos config,DEFAULT_GROUP,version = 2
```

不需要重启服务，不需要发送post请求，再次访问[http://localhost:3377/config/info]()，配置信息自动刷新了，得到以下结果：

```json
nacos config,DEFAULT_GROUP,version = 2
```

假设把配置文件中的prefix: config去掉，prefix默认值为${spring.application.name}的值，创建nacos-config-service-dev.yaml文件：

```yml
config:
    info: nacos config,DEFAULT_GROUP,version = 3333
```

此时，需要重启服务，重启后访问[http://localhost:3377/config/info](),得到以下结果：

```json
nacos config,DEFAULT_GROUP,version = 3333
```





#### 配置的隔离

配置文件这里有namespace、group、DataId这3个概念，用来区分不同的环境。

namespace是可以用来区分部署环境的，Group和DataId逻辑上区分两个目标对象。

**默认值：**

namespace=public，group=DEFAULT_GROUP，默认Cluster(集群)是DEFAULT

Nacos默认的命名空间是public，namespace主要是用来实现隔离。比如说我们现在有3个环境：开发，测试，生产环境，我们就可以创建三个namespace，不同的namaspace之间是隔离的。

Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组。



1. 默认namespace + DEFAULT_GROUP + 不同的DataId

通过不同的DataId(即不同的配置文件名称)，通过切换spring.profiles.active的值来切换不同的配置文件。

![image-DataId隔离配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221204841920.png)

```yml
spring:
  profiles:
#    active: dev # 加载nacos-config-service-dev.yaml配置文件
    active: prod # 加载nacos-config-service-prod.yaml配置文件
```



2. 默认的namespace + 相同的DataId + 不同的Group

![image-Group隔离配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221205245234.png)

通过配置文件中指定的group属性，切换不同的配置文件：

![image-通过group切换不同配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221205446832.png)





3. 不同的namespace + 相同的Group + 相同的DataId

![image-namespace隔离配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221205756914.png)

通过配置文件中指定的group属性，切换不同的配置文件：

![image-通过namespace切换不同配置文件](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221205920721.png)

![image-不同的namespace](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201221210119238.png)



#### Nacos集群

Nacos默认使用嵌入式数据库实现数据的存储。所以，如果启动多个配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySql的存储。



### Sentinel

#### 基本使用

1. 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2. 配置文件

```yml
server:
  port: 8401



spring:
  application:
    name: sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 47.96.224.198:8848
    sentinel:
      transport:
        # dashboard地址
        dashboard: 47.96.224.198:8080
management:
  endpoints:
    web:
      exposure:
        include: "*"

```

3. 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Sentinel8401Application {
    public static void main(String[] args) {
        SpringApplication.run(Sentinel8401Application.class, args);
    }
}

```

4. 业务类

```java
@RestController
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA() {
        return "------>testA";
    }

    @GetMapping("/testB")
    public String testB() {
        return "------>testB";
    }
}
```

5. 测试

sentinel默认是懒加载的，只有访问了url之后，页面才会有显示。

![image-sentinel控制台](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222200953755.png)



#### 流控规则

![image-新增流控规则](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222201255069.png)

上面的配置表示，当访问/testA时，每秒钟的访问量超过1（阈值），后面的请求就直接失败，快速失败默认的处理方式是抛出异常。

```json
Blocked by Sentinel (flow limiting)
```

这里有关QPS和线程数的说明：

* QPS：每秒的请求数量，和时间有关
* 线程数：跟线程的数量有关，当线程数量超过阈值，会进行限流

![image-线程数的配置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222203120210.png)



如果流控模式选择关联，当关联的资源达到阈值，就限流自己。

比如支付接口达到阈值，就要限流订单接口。

![image-关联](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222203702580.png)



warm up：根据codeFactory（冷加载因子，默认3）的值，从阈值codeFactory经过预热时长，才达到设置的QPS阈值。

应用场景：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是为了保护系统，可慢慢的把流量放进了，慢慢的把阈值增长到设置的阈值。

![image-预热](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222204548485.png)



排队等待：让请求以均匀的速度通过，阈值类型必须设置成QPS，否则无效。

应用场景：用于处理间隔性突发的流量，比如消息队列。在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

![image-排队等待](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201222205812768.png)





#### SentinelResource

使用@SentinelResource可以自定义异常处理方法。类似于Hystrix的@HystrixCommand.

@SententinelResource的value属性可以指定资源名称，在sentinel界面配置流控规则的时候，可以使用资源名称，也可以使用url。

![image-使用资源名称配置流控规则](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201223205817336.png)

使用blockHandler属性指明自定义的处理方法。当/testA(testA)被限流的时候，会进入到自定义的处理方法。自定义处理方法可以绑定一个BlockException。**标记@SentinelResource的方法形参有什么参数，自定义的处理方法也要有对应的形参。**

```java
@RestController
public class FlowLimitController {

    @GetMapping("/testA")
    @SentinelResource(value = "testA",blockHandler = "handlerException")
    public String testA() {

        return "------>testA";
    }

    public String handlerException(BlockException e) {
        return "自定义的处理方法...,发生错误:" + e;
    }
}
```

为了代码的耦合性低，可以将自定义方法写在一个类中，使用blockHandlerClass属性指明自定义方法所在的类，blockHandler指明要处理的自定义方法。

只要标明了blockHandlerClass属性，就会到对应的类中去查找自定义方法，即使本类中指定了同名的自定义方法也无效。

```java
@RestController
public class FlowLimitController {

    @GetMapping("/testA")
    @SentinelResource(value = "testA",blockHandlerClass = CustomerHandler.class,blockHandler = "handlerException")
    public String testA() {

        return "------>testA";
    }

    public String handlerException(BlockException e) {
        return "本类中的自定义的处理方法...,发生错误:" + e;
    }

}
```

CustomerHander是自定义所在方法的类。**类中的自定义处理方法必须使用static修饰。**

自定义处理方法类无需加入容器。

```java
public class CustomerHandler {

    public static String handlerException(BlockException e) {
        return "自定义类中的自定义的处理方法...,发生错误:" + e;
    }
}
```

如果在@SentinelResource内属性blockHandler没有识别到匹配的方法，会将属性fallback的内容替代blockHandler内容.

fallback指定的自定义方法上，异常对象绑定的是Throwable，如果绑定BlockException会报错。

```java
@RestController
public class FlowLimitController {

    @GetMapping("/testA")
    @SentinelResource(value = "testA",blockHandler = "handlerException111",fallback = "fallbackHandler")
    public String testA() {

        return "------>testA";
    }

    public String handlerException(BlockException e) {
        return "本类中的自定义的处理方法...,发生错误:" + e;
    }

    public String fallbackHandler(Throwable e) {
        return "如果在@SentinelResource内属性blockHandler没有识别到匹配的方法，会将属性fallback的内容替代blockHandler内容:"+ e;
    }
}
```



小结：

* **标记@SentinelResource的方法形参有什么参数，自定义的处理方法也要有对应的形参。**
* **类中的自定义处理方法必须使用static修饰。**
* **如果在@SentinelResource内属性blockHandler没有识别到匹配的方法，会将属性fallback的内容替代blockHandler内容.**

* **如果使用了@SentinelResource的value属性标记了资源名，流控规则不能配置为url的路径了。**
* **如果出现了message not variable的错误，很有可能的自定义方法的形参绑定错误了。**
* **流控规则不是持久化的，当重启了服务，配置的流控规则就会消失**
* **标记@SentinelResource的方法不能是private**



#### fallback和blockHandler

fallback指定的方法主要用来处理java中发生的异常，blockHandler指定的方法主要用来处理sentinel中规则的异常。



> 1. 只指定fallback

如果只指定了fallback方法，发生java异常会执行fallback指定的方法。

如果此时又违反了sentinel的规则，sentinel默认快速失败抛出的异常也会被fallback捕捉到进行处理。

```java
@RestController
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/payment/{id}")
    @SentinelResource(value = "test",fallback = "fallbackHandler")
    public String get(@PathVariable("id") String id) {
        return paymentService.get(id);
    }

    public String limitHandler(String id, BlockException e) {
        return "limitHandler处理sentinel配置的规则，e:" + e;
    }
}
```



> 2. 只指定blockHandler方法

如果出现java异常，没有fallback方法处理，会直接在浏览器页面抛出异常。

如果违反了sentinel配置的规则，会进入blockHandler指定的方法进行处理。

```java
@RestController
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/payment/{id}")
    @SentinelResource(value = "test",blockHandler = "limitHandler")
    public String get(@PathVariable("id") String id) {
        return paymentService.get(id);
    }

    public String limitHandler(String id, BlockException e) {
        return "limitHandler处理sentinel配置的规则，e:" + e;
    }
}
```



> 3. 指定fallback和blockhandler

出现的java异常通过fallback指定的方法处理，违反sentinel规则的通过blockHandler指定的方法处理。

但如果同时出现java异常和违反sentinel规则，则blockHandler指定的方法优先级高于fallback指定的方法。

```java
@RestController
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/payment/{id}")
    @SentinelResource(value = "test",fallback = "fallbackHandler",blockHandler = "limitHandler")
    public String get(@PathVariable("id") String id) {
        return paymentService.get(id);
    }


    public String fallbackHandler(String id, Throwable e) {
        return "fallbackHandler处理java中的异常，e:" + e;
    }

    public String limitHandler(String id, BlockException e) {
        return "limitHandler处理sentinel配置的规则，e:" + e;
    }
}

```



#### 持久化配置

默认情况下，服务重启后，之前配置的流控规则等就会消失，可以将其保存到nacos持久化保存。

1. 添加持久化依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

2. 添加配置

在配置文件中指明持久化的配置位于nacos配置中心的namespace、group、dataId等属性。

```yml
server:
  port: 9003

spring:
  application:
    name: sentinel-payment-service
  cloud:
    nacos:
      discovery:
        server-addr: 47.96.224.198:8848
    sentinel:
      transport:
        dashboard: localhost:8080
      # 持久化设置
      datasource:
        datasource1:
          nacos:
            server-addr: 47.96.224.198:8848
            dataId: sentinel-payment-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

3. 在nacos配置中心添加文件

![image-nacos持久化配置](C:\Users\zjm16\AppData\Roaming\Typora\typora-user-images\image-20201224223359921.png)



关于参数的介绍：

> resource:资源名称，也可以指定url的路径
>
> limitApp:来源应用
>
> grade:阈值类型，0表示线程数，1表示QPS
>
> count:单机阈值
>
> strategy:流控模式，0表示直接，1表示关联，2表示链路
>
> controlBehavior:流控效果，0表示快速失败，1表示Warm up，2表示排队等待
>
> clusterMode:是否集群



 

### seata

#### server端配置

1. 修改file.conf和registry.conf文件

在进行文件修改之前，先备份。

file.conf:

```shell
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis  存储模式修改为db
  mode = "db"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }
## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://localhost:3306/seata"  # 需要创建seata库，并导入sql语句，生成3张表
    user = "root"
    password = "123"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
## redis store property
  redis {
    host = "127.0.0.1"
    port = "6379"
    password = ""
    database = "0"
    minConn = 1
    maxConn = 10
    maxTotal = 100
    queryLimit = 100
  }

}

```

registry.conf:

```shell
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"  # 注册中心使用nacos
  loadBalance = "RandomLoadBalance"
  loadBalanceVirtualNodes = 10

  nacos {
    application = "seata-server"  # seata作为注册中心的一个服务，服务名称叫什么
    serverAddr = "localhost:8848" # nacos注册中心地址
    group = "SEATA_GROUP" # 服务分组，不是配置分组
    namespace = "" # 服务注册时的命名空间，不填为默认的public
    cluster = "default" # 不用改
    username = "nacos"
    password = "nacos"
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = 0
    password = ""
    cluster = "default"
    timeout = 0
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos" # 配置中心使用nacos

  nacos {
    serverAddr = "localhost:8848" # nasos配置中心的地址
    namespace = "" # 配置中心的命名空间
    group = "SEATA_GROUP" 
    username = "nacos"
    password = "nacos"
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
    apolloAccesskeySecret = ""
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}

```

2. 执行sql，导入seata的3张表

创建一个数据库seata，这与file.conf文件中的指定属性相同。

导入db_store.sql

```sql
-- the table to store GlobalSession data
DROP TABLE
IF EXISTS `global_table`;

CREATE TABLE `global_table` (
	`xid` VARCHAR (128) NOT NULL,
	`transaction_id` BIGINT,
	`status` TINYINT NOT NULL,
	`application_id` VARCHAR (32),
	`transaction_service_group` VARCHAR (32),
	`transaction_name` VARCHAR (128),
	`timeout` INT,
	`begin_time` BIGINT,
	`application_data` VARCHAR (2000),
	`gmt_create` datetime,
	`gmt_modified` datetime,
	PRIMARY KEY (`xid`),
	KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
	KEY `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
DROP TABLE
IF EXISTS `branch_table`;

CREATE TABLE `branch_table` (
	`branch_id` BIGINT NOT NULL,
	`xid` VARCHAR (128) NOT NULL,
	`transaction_id` BIGINT,
	`resource_group_id` VARCHAR (32),
	`resource_id` VARCHAR (256),
	`lock_key` VARCHAR (128),
	`branch_type` VARCHAR (8),
	`status` TINYINT,
	`client_id` VARCHAR (64),
	`application_data` VARCHAR (2000),
	`gmt_create` datetime,
	`gmt_modified` datetime,
	PRIMARY KEY (`branch_id`),
	KEY `idx_xid` (`xid`)
);

-- the table to store lock data
DROP TABLE
IF EXISTS `lock_table`;

CREATE TABLE `lock_table` (
	`row_key` VARCHAR (128) NOT NULL,
	`xid` VARCHAR (96),
	`transaction_id` LONG,
	`branch_id` LONG,
	`resource_id` VARCHAR (256),
	`table_name` VARCHAR (32),
	`pk` VARCHAR (36),
	`gmt_create` datetime,
	`gmt_modified` datetime,
	PRIMARY KEY (`row_key`)
);


```

3. 将配置文件导入nacos

config.txt是seata各种详细的配置，执行 nacos-config.sh 即可将这些配置导入到nacos，这样就不需要将file.conf和registry.conf放到我们client端的项目中了，需要什么配置就直接从nacos中读取。

config.txt:

```txt
transport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.thread-factory.boss-thread-prefix=NettyBoss
transport.thread-factory.worker-thread-prefix=NettyServerNIOWorker
transport.thread-factory.server-executor-thread-prefix=NettyServerBizHandler
transport.thread-factory.share-boss-worker=false
transport.thread-factory.client-selector-thread-prefix=NettyClientSelector
transport.thread-factory.client-selector-thread-size=1
transport.thread-factory.client-worker-thread-prefix=NettyClientWorkerThread
transport.thread-factory.boss-thread-size=1
transport.thread-factory.worker-thread-size=8
transport.shutdown.wait=3
# nacos配置中心和springboot中的值都需要和这里一致
service.vgroup_mapping.my_test_tx_group=default
service.enableDegrade=false
service.disable=false
service.max.commit.retry.timeout=-1
service.max.rollback.retry.timeout=-1
client.async.commit.buffer.limit=10000
client.lock.retry.internal=10
client.lock.retry.times=30
client.lock.retry.policy.branch-rollback-on-conflict=true
client.table.meta.check.enable=true
client.report.retry.count=5
client.tm.commit.retry.count=1
client.tm.rollback.retry.count=1
store.mode=file
store.file.dir=file_store/data
store.file.max-branch-session-size=16384
store.file.max-global-session-size=512
store.file.file-write-buffer-cache-size=16384
store.file.flush-disk-mode=async
store.file.session.reload.read_size=100
store.db.datasource=dbcp

store.db.datasource=druid
store.db.db-type=mysql
store.db.driver-class-name=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true
store.db.user=root
store.db.password=123

store.db.min-conn=1
store.db.max-conn=3
store.db.global.table=global_table
store.db.branch.table=branch_table
store.db.query-limit=100
store.db.lock-table=lock_table
recovery.committing-retry-period=1000
recovery.asyn-committing-retry-period=1000
recovery.rollbacking-retry-period=1000
recovery.timeout-retry-period=1000
transaction.undo.data.validation=true
transaction.undo.log.serialization=jackson
transaction.undo.log.save.days=7
transaction.undo.log.delete.period=86400000
transaction.undo.log.table=undo_log
transport.serialization=seata
transport.compressor=none
metrics.enabled=false
metrics.registry-type=compact
metrics.exporter-list=prometheus
metrics.exporter-prometheus-port=9898
support.spring.datasource.autoproxy=false

```

使用nacos-config.sh将配置导入到nacos的配置中心。

如果nacos-config.sh不是可执行文件，需要增加权限 chmod 777 nacos-config.sh

config存放的目录位于nacos-config.sh的上一级目录。

* config.txt : /usr/local/seata/seata
* config-config.sh : /usr/local/seata/seata/conf

```shell
sh nacos-config.sh -h localhost -p 8848 -g SEATA_GROUP -t 7ff58f98-79e2-41ad-b96e-a2721c7864af
```

参数说明：

-h: host，默认值 localhost

-p: port，默认值 8848

-g: 配置分组，默认值为 'SEATA_GROUP'

-t: 租户信息，对应 Nacos 的命名空间ID字段, 默认值为空 ''

4. 创建logs文件夹

在/usr/local/seata/seata目录下创建文件夹logs，并在logs下创建seata_gc.log文件

![image-logs文件夹](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201228214652142.png)

5. 启动seata

在bin目录下./seata-server.sh启动。

如果启动的时候报内存不足的错误，可以修改seata-server.sh文件，修改其中一行的配置。

```shell
exec "$JAVACMD" $JAVA_OPTS -server -Xmx256m -Xms256m -Xmn256m -Xss512k 
```

出现以下提示说明启动成功：

![image-seata启动](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201228214951974.png)



此时，查看nacos：

![image-nacos中查看seata服务](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201228215112833.png)

registry.conf的其中一段配置：

```shell
nacos {
    application = "seata-server"
    serverAddr = "localhost:8848"
    group = "SEATA_GROUP"
    namespace = "7ff58f98-79e2-41ad-b96e-a2721c7864af"
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }

```

查看nacos的配置中心：

![image-nacos查看seata的配置](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20201228215401610.png)





#### client端配置

1. 需要使用seata的客户端需要引入依赖，版本依赖于父级POM

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2.  导入undo_log





3. springboot配置文件



```yml
seata:
  enabled: true
  enable-auto-data-source-proxy: true
  tx-service-group: my_test_tx_group # 事务群组（可以每个应用独立取名，也可以使用相同的名字）
  client:
    rm-report-success-enable: true
    rm-table-meta-check-enable: false # 自动刷新缓存中的表结构（默认false）
    rm-report-retry-count: 5 # 一阶段结果上报TC重试次数（默认5）
    rm-async-commit-buffer-limit: 1000 # 异步提交缓存队列长度（默认10000）
    rm:
      lock:
        lock-retry-internal: 10 # 校验或占用全局锁重试间隔（默认10ms）
        lock-retry-times:    30 # 校验或占用全局锁重试次数（默认30）
        lock-retry-policy-branch-rollback-on-conflict: true # 分支事务与其它全局回滚事务冲突时锁策略（优先释放本地锁让回滚成功）
    tm-commit-retry-count:   3 # 一阶段全局提交结果上报TC重试次数（默认1次，建议大于1）
    tm-rollback-retry-count: 3 # 一阶段全局回滚结果上报TC重试次数（默认1次，建议大于1）
    undo:
      undo-data-validation: true # 二阶段回滚镜像校验（默认true开启）
      undo-log-serialization: jackson # undo序列化方式（默认jackson）
      undo-log-table: undo_log  # 自定义undo表名（默认undo_log）
    log:
      exceptionRate: 100 # 日志异常输出概率（默认100）
    support:
      spring-datasource-autoproxy: true # 数据源自动代理开关（默认false关闭）
  service:
    vgroup-mapping:
      my_test_tx_group: default # TC 集群（必须与seata-server保持一致）
    enable-degrade: false # 降级开关
    disable-global-transaction: false # 禁用全局事务（默认false）
  transport:
    enable-client-batch-send-request: true # 客户端事务消息请求是否批量合并发送（默认true）
  # 注册中心
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 47.96.224.198
      username: nacos
      password: nacos
  # 配置中心
  config:
    type: nacos
    nacos:
      server-addr: 47.96.224.198:8848
      group: SEATA_GROUP
      username: nacos
      password: nacos
      namespace: 7ff58f98-79e2-41ad-b96e-a2721c7864af

```



方法一 你可以通过file.conf来配置使用环境
file.conf如以下修改
vgroup_mapping.xxxx-fescar-service-group = "default"
xxx是你的application名

方法二 通过环境变量改变使用环境配置
在启动脚本追加以下命令
export SEATA_CONFIG_ENV=default

registry.conf 对应的是 default
registry-dev.conf 对应 dev开发环境
registry-test.conf 对应 test测试环境
registry-prod.conf 对应 prod生产环境