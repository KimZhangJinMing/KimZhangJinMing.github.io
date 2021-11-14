---
title: SpringBoot日志.md
date: 2020-08-10 22:46:35
categories: SpringBoot
tags: SpringBoot日志
---

### SpringBoot使用的日志框架

* SpringBoot使用的是slf4j + logback日志实现
* 导入其他的包将其他组件的日志实现转换成slf4j



### 使用日志

```java
Logger logger = LoggerFactory.getLogger(getClass());
// 日志的级别由低到高
logger.trace("trace level");
logger.debug("debug level");
logger.info("info level");
logger.warn("warn level");
logger.error("error level");
```

SpringBoot默认的日志级别是INFO，只会输出级别大于info的日志信息。可以在配置文件中设置日志级别。

```properties
// 调整全局的日志级别
logging.level.root=trace

// 调整某个包的日志级别
logging.level.com.sise.jpa=trace
```

### 指定日志配置文件位置

在配置文件中可以指定日志输出的位置：

| logging.file.name | logging.file.path | 描述                                                         |
| ----------------- | ----------------- | ------------------------------------------------------------ |
| [none]            | [none]            | 只在控制台输出                                               |
| 指定文件名        | [none]            | 输出日志到指定文件名中，不指定路径在当前项目下生成指定文件名的文件 |
| [none]            | 指定目录          | 输出到指定目录的spring.log文件中                             |
| 指定文件名        | 指定文件夹        | 输出日志到指定文件名中                                       |

```properties
// 日志输出存放在当前项目目录下，文件名称为springboot.log
logging.file.name=springboot.log

// 日志输出存放到指定目录下的指定文件名
logging.file.name=C:\\Users\\zjm16\\Desktop\\springboot.log

// 日志输出存放在当前项目下的指定目录下
logging.file.name=spring/log/springboot.log

// 日志输出存放在指定的文件夹，文件名称为默认的spring.log
logging.file.path=spring/log
```

总结:

logging.file.name 可以指定路径和log文件的名字

logging.file.path 只可以只当log的路径, 不能指定log的名字, 使用缺省值spring.log

二者只可以存在一个

### 