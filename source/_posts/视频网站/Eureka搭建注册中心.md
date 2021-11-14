### 1. 搭建工程

#### 1.1 报错信息

如果@EnableEurekaServer找不到,可能是因为maven版本冲突的原因。

如使用SpringBoot2.2.2.RELEASE版本,SpringCloud的版本是Hoxton.RELEASE,eureka-server的版本应该是2.2.2.RELEASE,而不是自动导入的2.2.0.RELEASE.



如果启动报错以下信息:

```java
DiscoveryClient_EUREKASERVER/ming:EurekaServer:8001 - was unable to refresh its cache! status = Cannot execute request on any known server
```

启动服务后报错，添加以下配置：

```yml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```



#### 1.2 mybatis-generator

首先,在父POM中添加mybatis-generator的插件,并添加mysql的驱动作为依赖:

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    <!-- 指定generator配置文件的路径 -->
                    <configurationFile>src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
               	<!-- 添加mysql驱动 -->
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${mysql.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```

在src/main/resources/generator/generatorConfig.xml添加配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="mysql" targetRuntime="MyBatis3" defaultModelType="flat">
        
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
    
        <!--生成的实体类自带toString方法-->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--覆盖xml-->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin"/>
        <commentGenerator>
            <!--<property name="suppressDate" value="false"/>-->
            <!--不生成注释-->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        
       
        
       
        
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/cloud-course" userId="root" password="123"/>
        
        <!--实体类-->
        <javaModelGenerator targetPackage="com.imooc.domain"
                            targetProject="src\main\java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        
        <!--mapper.xml-->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src\main\resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        
        <!--mapper接口-->
        <javaClientGenerator targetPackage="com.imooc.mapper"
                             targetProject="src\main\java" type="XMLMAPPER">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        
        <table tableName="user" domainObjectName="User"/>
    </context>
</generatorConfiguration>
```

1. 配置文件中property,plugin,commentGenerator是有顺序的,如果顺序没有按照规定的来,会报错`元素类型为 "context" 的内容必须匹配`
2. targerProject中要使用\,而不是/

添加完配置文件后,有两种使用方法:

1.在Maven的命令行中执行：

![在maven命令中使用](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210424203023538.png)

2.配置一个maven命令:

![配置一个maven命令](https://jinming8.oss-cn-shenzhen.aliyuncs.com/img/image-20210424203153345.png)