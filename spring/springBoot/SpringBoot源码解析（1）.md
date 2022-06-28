# SpringBoot源码解析（1）

​		Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。今天我们将基于SpringBoot 2.6.3从头开始逐行解析SpringBoot的源码实现。

​		首先一个简单的SpringBoot项目，通常要有如下代码

```java
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

​		其中，最值得关注的就是SpringApplication.run()方法和@SpringBootApplication注解，首先让我们先来看看一下SpringApplication.run()都做了一些什么。

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args){
	return run(new Class<?>[] { primarySource }, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}
```

​		首先方法调用的SpringApplication的静态助手，通过默认配置和用户传递的方法来运行SpringApplication。程序首先初始化了一个对象，我们里看一下他的构造方法做了写什么。

```java
public SpringApplication(Class<?>... primarySources) {
	this(null, primarySources);
}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    //设置资源加载器
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
    //设置主配置类
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	this.bootstrapRegistryInitializers = new ArrayList<>(
				getSpringFactoriesInstances(BootstrapRegistryInitializer.class));			setInitializers((Collection)getSpringFactoriesInstances(
                                     ApplicationContextInitializer.class));
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
}
```

​		接下来我们来分析WebApplicationType.deduceFromClasspath()方法。这个方法是用来推断web环境的。这里面定义了3种不同的web环境 

​		NONE：不启动内嵌的WebServer，不是运行web application

​		SERVLET：启动内嵌的基于servlet的web server

​		REACTIVE：启动内嵌的reactive web server

```java
private static final String[] SERVLET_INDICATOR_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" };

	private static final String WEBMVC_INDICATOR_CLASS = 	"org.springframework.web.servlet.DispatcherServlet";

	private static final String WEBFLUX_INDICATOR_CLASS = "org.springframework.web.reactive.DispatcherHandler";

	private static final String JERSEY_INDICATOR_CLASS = "org.glassfish.jersey.servlet.ServletContainer";

	private static final String SERVLET_APPLICATION_CONTEXT_CLASS = "org.springframework.web.context.WebApplicationContext";

	private static final String REACTIVE_APPLICATION_CONTEXT_CLASS = "org.springframework.boot.web.reactive.context.ReactiveWebApplicationContext";

static WebApplicationType deduceFromClasspath() {
	if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && 	     							!ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null) &&									!ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
	for (String className : SERVLET_INDICATOR_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	return WebApplicationType.SERVLET;
}

public static boolean isPresent(String className, @Nullable ClassLoader classLoader) {
		try {
			forName(className, classLoader);
			return true;
		}
		catch (IllegalAccessError err) {
			throw new IllegalStateException("Readability mismatch in inheritance hierarchy of class [" +
					className + "]: " + err.getMessage(), err);
		}
		catch (Throwable ex) {
			// Typically ClassNotFoundException or NoClassDefFoundError...
			return false;
		}
	}
```

​		他的核心源码其实很简单，就是根据类路径去加载对应的类。它的底层调用的是java.lang.Class#forName(java.lang.String, boolean, java.lang.ClassLoader)。这个方法会加载你指定的className到jvm。如果加载失败获取不到类的话就会抛出异常，返回flase。然后根据类的加载情况来判断当前的web环境。那么什么情况下会加载不到类呢？其实就是你没有导入对应的包，所以其实就是根据你导入的包，来确定web环境。待续。。。





