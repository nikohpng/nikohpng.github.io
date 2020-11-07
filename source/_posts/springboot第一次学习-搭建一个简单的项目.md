---
title: springboot第一次学习-搭建一个简单的项目
date: 2020-09-01 21:53:50
cover: /images/springboot.jpg
tags: maven
category: springboot

---

阅读官方文档，官方文档中对于springboot的简单项目搭建的基本解释。

## `SpringBoot`中`spring-parent`的作用

> Spring Boot provides a number of “Starters” that let you add jars to your classpath. Our applications for smoke tests use the `spring-boot-starter-parent` in the `parent` section of the POM. The `spring-boot-starter-parent` is a special starter that provides useful Maven defaults. It also provides a [`dependency-management`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-dependency-management) section so that you can omit `version` tags for “blessed” dependencies.

+ 根据文档的翻译可以简单概括为spring-boot-starter-parent不提供依赖关系。它主要提供maven的一些默认值，一个依赖管理器用于你添加依赖的时候不需要在maven中加version。[原文链接](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-first-application-dependencies)
+ <span style="color:red;">注意：其中不提供依赖的意思是，它不提供如start-web这一类的web启动依赖。即spring-boot-starter-parent不是启动程序，它只是提供了管理工具与预设置值</span>

## `SpringBoot`的`Plugin`

```xml
<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
```

`Plugin`有7个features:

- Java 1.8 as the default compiler level.

- UTF-8 source encoding.

- A dependency management section, inherited from the `spring-boot-dependencies` POM, that manages the versions of common dependencies. This dependency management lets you omit `` tags for those dependencies when used in your own POM.

- An execution of the [`repackage` goal](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/#goals-repackage) with a `repackage` execution id.

- Sensible [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html).

- Sensible plugin configuration ([Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin), and [shade](https://maven.apache.org/plugins/maven-shade-plugin/)).

- Sensible resource filtering for `application.properties` and `application.yml` including profile-specific files (for example, `application-dev.properties` and `application-dev.yml`)

  > Since the `application.properties` and `application.yml` files accept Spring style placeholders (`${…}`), the Maven filtering is changed to use `@..@` placeholders. (You can override that by setting a Maven property called `resource.delimiter`.)

`SpringBoot`的`Plugin`有以下7个作用,其中原文如上，下方是个人理解

+ `java8`作为默认的编译器

+ `UTF-8`作为打包时候的编码方式

+ 从spring启动依赖项POM继承的依赖项管理部分，用于管理公共依赖项的版本。这种依赖项管理允许您在自己的POM中使用这些依赖项时省略<version>标记。

+ 不懂啥意思，直译好像是重新打包的时候会伴随一个重新打包的id

+ `SpringBoot`中打包时候传递参数给`application.yml`配置文件，对其进行资源过滤

+ `SpringBoot`中打包时候修改配置`git commit id`和`shade`

+ `SpringBoot`中打包时根据需求过滤不同形式的文件类型，如 `application.properties` and `application.yml`

  > 简单来说就是从maven中的pom.xml传递参数到application.properties时，需要使用@..@引用变量。如果想修改的话，在pom.xml中的properties中通过<resource.delimiter>@<resource.delimiter>进行修改

## `SpringBoot`在`POM`中配置示例

### `SpringBoot`作为父项目

```xml
<xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <!--设置项目的默认参数与定义环境变量-->
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
	
    <!--继承springboot作为当前项目的父项目-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.5.RELEASE</version>
    </parent>
    
     <!--具体的依赖关系-->
    <dependencies>
    </dependencies>
    
    <!--引入springboot的管理插件-->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

### `SpringBoot`不作为父项目

由于在公司开发的时候可能需要继承公司的父项目，因而无法继承`SpringBoot`作为父项目。可采用以下形式

```xml

<xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <!--设置项目的默认参数与定义环境变量-->
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  	</properties>
    
    <!--引入springboot，dependencyManagement只是声明依赖，不引入-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.5.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <!--具体的依赖关系-->
    <dependencies>
    </dependencies>
    
    
    <!--引入springboot的管理插件-->
    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
 </project>
```





