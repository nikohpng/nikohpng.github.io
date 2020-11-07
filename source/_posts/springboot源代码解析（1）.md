---
title: springboot源代码解析（1）
date: 2020-10-31 15:40:36
cover: /images/springboot.jpg
tags: spring boot源码
category: springboot
---

主要对spring boot的源代码进行解析，解析的内容包括启动顺序、代码的作用和设计代码的设计模式。第一次内容目标需要解决的问题关于springboot的配置的加载！

## Spring Boot的入口类

```java
@SpringBootApplication
public class SpringBootBestPracticeApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootBestPracticeApplication.class, args);
	}
	
}
```

以上入口类主要使用`SpringApplication`中的方法，跟踪代码进入`run`

```java
public static ConfigurableApplicationContext run(Class<?> primarySource,
			String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}

/**
	静态助手，可用于使用默认设置和用户提供的参数从指定的源运行SpringApplication。
**/
public static ConfigurableApplicationContext run(Class<?>[] primarySources,
			String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
```

函数上的注释为函数的英文注释的中文翻译。

其中第一个参数`primarySource`:加载主要资源类（未知用途）

第二个参数`args`:用于传递参数（未知用途）

**设计模式：**采用了静态工厂方法，其中主要让创建对象的时候返回其它类型的对象且具有可读性。---<<Effective-Java>>

## `SpringApplication`实例化

```java
public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    	//初始化资源加载器为null
		this.resourceLoader = resourceLoader;
    	
    	//断言primarySources是否为null,不能为null
		Assert.notNull(primarySources, "PrimarySources must not be null");
    
    	//初始化加载资源类集合并去重
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    
    	//推断web类型，SERVLET，REACTIVE，None
		this.webApplicationType = deduceWebApplicationType();
    
    	//设置应用上下文初始化器
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
    	
    	//设置应用监听器
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    
    	//推断主入口应用类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

`SpringApplication`实例化过程中经过了`7`个步骤，前`3`个步骤集中于对`resourceLoader`、`primarySources`的初始与判定，较为简单。后四个步骤为实例化的重点

### 推断`web`类型

```java
private WebApplicationType deduceWebApplicationType() {
    
    //判断如果存在*reactive.DispatcherHandler
    //并且不存在*web.servlet.DispatcherServlet
    //并且不存在org.glassfish.jersey.server.ResourceConfig
		if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
    //判断是否存在javax.servlet.Servlet和
    //*.ConfigurableWebApplicationContext
		for (String className : WEB_ENVIRONMENT_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}

public enum WebApplicationType {

	/**
	 * The application should not run as a web application and should not start an
	 * embedded web server.
	 */
	NONE,

	/**
	 * The application should run as a servlet-based web application and should start an
	 * embedded servlet web server.
	 */
	SERVLET,

	/**
	 * The application should run as a reactive web application and should start an
	 * embedded reactive web server.
	 */
	REACTIVE

}
```

`ClassUtils.isPresent`函数主要用于确定类是否存在

`WebApplicationType`有三种类型，`None`表示不是以一个web应用启动，`servlet`表示以`servlet`为基础的web应用，`REACTIVE`表示是一个reactive的web。`REACTIVE`本人研究不多，按照`Spring`官方说法是一个非阻塞的web框架，其使用到的maven为`spring-boot-starter-webflux`

### 设置应用上下文初始化器

```java
setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
```

#### 应用上下文初始化器的作用

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

	/**
	 * Initialize the given application context.
	 * @param applicationContext the application to configure
	 */
	void initialize(C applicationContext);

}
```

用来初始化指定的 Spring 应用上下文，如注册属性资源、激活配置文件等

#### 设置应用上下文初始化

```java
public void setInitializers(
			Collection<? extends ApplicationContextInitializer<?>> initializers) {
		this.initializers = new ArrayList<>();
		this.initializers.addAll(initializers);
	}
```

只是将其初始化一个名为`initializers`的集合

#### `getSpringFactoriesInstances`的作用

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}

	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
        
        //获取当前线程的类加载器
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
        //获取ApplicationContextInitializer实例类并去重
		Set<String> names = new LinkedHashSet<>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        //根据实例类实例化对应并进行排序
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```





