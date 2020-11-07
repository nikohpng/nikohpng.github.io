---
title: Spring Boot第二次学习-关于Spring Boot的日志配置
date: 2020-09-06 09:56:25
cover: /images/logback.jpg 
tags: log
category: springboot

---

阅读官方文档，明确Spring Boot采用的日志系统形式与共存关系，Spring Boot日志的高级配置，Spring Boot日志文件系统的依赖关系，日志接口与日志实现对Spring Boot日志输出的影响简谈，lombok与Spring Boot的结合使用

## 一、Spring Boot采用日志系统形式与共存关系

> Spring Boot uses [Commons Logging](https://commons.apache.org/logging) for all internal logging but leaves the underlying log implementation open. Default configurations are provided for [Java Util Logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html), [Log4J2](https://logging.apache.org/log4j/2.x/), and [Logback](https://logback.qos.ch/). In each case, loggers are pre-configured to use console output with optional file output also available.
>
> By default, if you use the “Starters”, Logback is used for logging. Appropriate Logback routing is also included to ensure that dependent libraries that use Java Util Logging, Commons Logging, Log4J, or SLF4J all work correctly.

+ 上诉内容讲述了四个方面

  + 第一：Spring Boot默认采用Commons Logging作为日志接口

  + 第二：Spring Boot默认配置提供[Java Util Logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html), [Log4J2](https://logging.apache.org/log4j/2.x/), 和[Logback](https://logback.qos.ch/)

  + 第三：如果你使用了"Starters"，那么Logback会作为默认的日志实现

  + 第四：Spring Boot中，在你采用了Logback的情况下，任然可以使用Java Util Logging, Commons Logging, Log4J, 或者 SLF4J。

    >这句话有两层含义：
    >
    >其一，在使用Logback作为日志接口实现的情况，仍然可以使用其它日志实现（eg:log4j）。可查看第四节第二点
    >
    >其二，在默认使用的Commons Logging作为日志接口的情况下，任然可以使用SLF4J作为日志接口。可查看第四节第一点

+ 总结：Spring Boot默认使用Commons Logging与Logback作为日志文件系统组合。在使用默认日志系统的情况，仍然可以使用其它的日志系统接口与日志系统实现。

## 二、Spring Boot日志的高级配置

+ <span style="color:red;">注:</span>为简化问题，此处选择兼容性较好的SLF4J与Logback作为演示基础

### 2.1、application.yaml配置方式

+ 文件名与文件路径

  | logging.file.path | logging.file.name | 示例                    | 描述                               |
  | ----------------- | ----------------- | ----------------------- | ---------------------------------- |
  | 无                | 无                |                         | 只在控制台输出                     |
  | 无                | 配置文件名        | my.log或/var/log/my.log | 输出到特定的文件中。               |
  | 配置路径          | 无                | /var/log                | 输出到文件夹中，文件名为spring.log |
  
  > tips :Logback独有的日志配置，无法在application.yaml中配置。这里只能配置一些基础内容。name与path不能同时使用，就算同时使用也只会有`logging.file.path`生效
  
+ 其它文件属性配置参数

  + 日志文件的最大大小：默认为10M,可通过`logging.file.max-size`配置。

  + 日志文件存储的时间：默认存储最近7天，可通过`logging.file.max-history`配置。

  + 日志文档的总大小：当日志文档总大小超过该阈值会删除备份，可通过`logging.file.total-size-cap`。

  + 强制启动时清理日志：`logging.file.clean-history-on-start`。

  + 控制台日志输出过滤：`logging.pattern.console`。

  + 日志的日期格式：`logging.pattern.dateformat`

  + 过滤日志等级：`logging.pattern.level`

  + 日志滚动的名字：`logging.pattern.rolling-file-name`，默认的名字为`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`

    > tips: 仅支持Logback

+ 日志的等级

  + 配置全局的日志等级：`logging.level.root`
  + 配置包的日志等级：`logging.level.包名`(eg:`logging.level.org.hibernate`）

+ 日志分组

  + 解释：通过日志分组，可以同时修改多个包的日志输出等级
  + 分组：`logging.group.分组名`（eg : `**logging.group.tomcat**=org.apache.catalina, org.apache.coyote, org.apache.tomcat`）
  + 修改输出等级：`logging.level.分组名`（eg : `logging.level.tomcat`）

  > tips: Spring Boot中带有默认的日志分组，可前往官网自行查看，[日志分组](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-custom-log-groups)

### 2.2、日志的xml配置方式

#### 2.2.1、Spring Boot加载xml

+ 修改配置，指定xml位置:`logging.config`可以配置加载xml路径

+ 默认加载配置

  | 日志系统                | 配置方式                                                     |
  | ----------------------- | ------------------------------------------------------------ |
  | Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, 或`logback.groovy` |
  | Log4j2                  | `log4j2-spring.xml` 或 `log4j2.xml`                          |
  | JDK (Java Util Logging) | `logging.properties`                                         |

  

#### 2.2.2、Spring Boot中配置logback-spring.xml

+ <span style="color:red">注:</span> 本文不关注logback的xml中的具体内容，因而可自行百度查询各个字段含义

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <substitutionProperty name="log.base" value="F:\\TestImage" />

    <!--控制台输出-->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date [%thread][IP:%X{ip}|USER:%X{user}][%-5level %logger{80}] %msg%n</pattern>
        </encoder>
    </appender>

    <!--滚动日志：其中包括日志输出名字、滚动的规则-->
    <appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.base}/stdout.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${log.base}/stdout.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- or whenever the file size reaches 100MB -->
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date [%thread][IP:%X{ip}|USER:%X{user}][%-5level %logger{80}] %msg%n</pattern>
        </encoder>
    </appender>

    <!--异常日志输出：其中包括日志输出名、滚动规则-->
    <appender name="ExceptionLoggerFileOut" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.base}/exception.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${log.base}/exception.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- or whenever the file size reaches 100MB -->
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date [%thread][IP:%X{ip}|USER:%X{user}][%-5level %logger{80}] %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.shangde.common.sys.log.SysLogInterceptor" level="DEBUG" additivity="true" >
        <appender-ref ref="ExceptionLoggerFileOut" />
    </logger>

    <!--各个包的日志输出等级-->
    <logger name="com.cff" level="DEBUG" />
    <logger name="com" level="INFO" />
    <logger name="ch.qos.logback" level="INFO" />
    <logger name="org.mybatis" level="INFO" />
    <logger name="org" level="INFO" />
    <logger name="springfox" level="INFO" />

    <!--总的输出等级-->
    <root>
        <level value="DEBUG" />
        <appender-ref ref="stdout" />
        <appender-ref ref="rollingFile" />
    </root>
</configuration>
```

#### 2.2.3、Spring Boot中配置log4j2-spring.xml

+ <span style="color:red">注:</span> 本文不关注logback的xml中的具体内容，因而可自行百度查询各个字段含义

+ 使用log4j2作为Spring Boot的日志系统，需要先将"Starts"中的Logback排除，增加Log4j2的Spring Boot包。**日志接口仍然采用SLF4J！**

  ```xml
   <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
              <exclusions>
                  <exclusion>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-logging</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-log4j2</artifactId>
          </dependency>
  ```

+ xml配置文件内容

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <Configuration status="OFF" monitorInterval="1800">
      <properties>
          <property name="LOG_HOME">D:/test</property>
          <property name="FILE_NAME">stdout</property>
      </properties>
      <Appenders>
          <Console name="Console" target="SYSTEM_OUT">
              <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
          </Console>
  
          <RollingFile name="running-log" fileName="${LOG_HOME}/${FILE_NAME}.log"
              filePattern="${LOG_HOME}/$${date:yyyy-MM}/${FILE_NAME}-%d{yyyy-MM-dd}-%i.log.gz"
              immediateFlush="true">
              <PatternLayout
                  pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n" />
              <Policies>
                  <TimeBasedTriggeringPolicy />
                  <SizeBasedTriggeringPolicy size="10 MB" />
              </Policies>
              <DefaultRolloverStrategy max="20" />
          </RollingFile>
      </Appenders>
      <Loggers>
          <Logger name="com.cff" level="debug" additivity="true">
              <AppenderRef ref="running-log" />
              <AppenderRef ref="Console" />
          </Logger>
          <Root level="info">
              <!-- 这里是输入到文件，很重要 -->
              <AppenderRef ref="running-log" />
              <!-- 这里是输入到控制台 -->
              <AppenderRef ref="Console" />
          </Root>
      </Loggers>
  </Configuration>
  ```

  

#### 2.2.4、关于log4j的特殊说明

+ 根据文中的意思，log4j日志系统具有重大缺陷，因而建议将日志系统切换为log4j2或logback。[原文内容](https://mp.weixin.qq.com/s/wAqmsghhmiTDbv1GGFHTKQ)

## 三、Spring Boot日志文件系统的依赖关系

+ Logback是"Start"中默认引入的日志系统

  ![image][logback]

+ Log4J2需要单独引入，引入时需要排除Logback

  ```xml
   <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
              <exclusions>
                  <exclusion>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-logging</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-log4j2</artifactId>
          </dependency>
  ```

## 四、日志接口与日志实现对Spring Boot日志输出的影响简谈

+ 在Spring Boot中使用Commons Logging或SLF4J，那么实现可以随意使用Logback或Log4J,影响日志输出的将只与它们的配置方式有关，Spring Boot的配置方式可查看**第三章**。其中关于Log4J的使用可参考上文。关于Logback或Log4J对于Spring Boot日志输出的影响，可看下表：

  | 日志系统 | yaml配置                 | xml配置            | 解释与结论                                                   |
  | -------- | ------------------------ | ------------------ | ------------------------------------------------------------ |
  | Logback  | 不配置                   | log4j2-spring.xml  | 由于引入的Logback日志系统，配置将以YAML中的配置为准，即只输出到控制台 |
  | Logback  | 不配置                   | logback-spring.xml | 由于引入Logback日志系统，配置将以logback.xml中的配置为准     |
  | Log4J    | 不配置                   | logback-spring.xml | 由于引入的Log4J日志系统，配置将以YAML中的配置为准，即只输出到控制台 |
  | Log4J    | 不配置                   | log4j2-spring.xml  | 由于引入Log4J日志系统，配置将以log4j2.xml中的配置为准        |
  | Logback  | 配置xml文件为log4j2.xml  | log4j2-spring.xml  | 将以yaml中配置的xml文件路径为准，日志输出以xml中内容为准     |
  | Log4J    | 配置xml文件为logback.xml | logback-spring.xml | 将以yaml中配置的xml文件路径为准，日志输出以xml中内容为准     |

  > tips : 日志接口Commons Logging或SLF4J可以同时使用

+ 在Spring Boot程序运行过程中，通过使用`@log4j2、@log等`这种具体的日志系统而不是日志系统接口

  + 注：这种情况下，只要在maven中引入了对应的日志系统实现包就可以在Spring Boot中使用，并且任然可以打印到控制台，只是可能存在无法输出到文件中。具体原因是你Spring Boot使用的是Logback日志系统，你单独使用的是Log4j2，那么将无法输出到文件中。

## 五、Spring Boot与lombok的结合使用

### 5.1、lombok在IDEA与Maven中的安装

+ 本人使用的是Idea，因而在Plugins中搜索lombok，安装对应的插件。
+ 在Maven引入lombok的依赖

### 5.2、lombok在IDEA与Maven的作用

+ Plugins中的lombok作用：是防止IDEA的自动语法检查导致的报错。原因是在maven中包引入后，lombok会在编译期间对代码进行修改，但是IDEA并不知道这些修改（eg:getValue()方法会错误），因而需要引入lombok的插件

+ Maven中的lombok作用：实现JSR269api,修改AST(抽象语法树)

  > tips:上诉内容属于lombok的实现原理，具体内容我参考了这篇博客[《lombok原理分析与功能实现》](https://www.jianshu.com/p/fc06578e805a)

### 5.3、lombok的使用（日志方面）

+ @**CommonsLog**

  ```java
  private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);
  ```

+ **@Flogger**

  ```java
  private static final com.google.common.flogger.FluentLogger log = com.google.common.flogger.FluentLogger.forEnclosingClass();
  ```

+ **@JBossLog**

  ```java
  private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);
  ```

+ **@Log**

  ```java
  private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
  ```

+ **@Log4j**

  ```java
  private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);
  ```

+ **@Log4j2**

  ```java
  private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
  ```

+ **@Slf4j**

  ```java
  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
  ```

+ **@XSlf4j**

  ```java
  private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
  ```

+ **@CustomLog**

  ```java
  private static final com.foo.your.Logger log = com.foo.your.LoggerFactory.createYourLogger(LogExample.class);
  ```

  > tips:主要的使用内容，可点击[lombok日志使用](https://projectlombok.org/features/log)

[logback]:data:image/png;base64,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