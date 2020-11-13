# 启动类
```java
SpringBoot启动类的核心就是两个 一注解一方法
@SpringBootApplication
SpringApplication.run(Application.class, args);
只需要研究好这两个就可以了
总结：
@SpringBootApplication实际上是一个复合注解,为了简化书写其实际上包含了几个核心注解
1. @SpringBootApplication
2. @ComponentScan
3. @EnableAutoConfiguration
其中@SpringBootApplication注解继承的是@configuration注解其实际作用等于xml中的beans即启动spring容器管理spring容器中的bean，通常和@bean用于启动容器并配置bean或者@componentScan一起使用启动容器并扫描所在包下的所有组件并注册到容器中。
@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前创建并使用的IoC容器。

SpringApplication.run一共做了两件事,一件是创建SpringApplication对象,在对象初始化时保存事件监听器，容器初始化类以及判断是否为web应用，保存包含main方法的主配置类。第二件事就是运行run方法,准备spring的上下文，完成容器的初始化，创建，加载等。会在不同的时机触发监听器的不同事件。

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
# @SpringBootApplication

> @SpringBootApplication = (默认属性)@Configuration + @EnableAutoConfiguration + @ComponentScan。

```java
@SpringBootApplication它是一个组合注解，重点介绍一下如下三个

@Configuration（@SpringBootConfiguration点开查看发现里面还是应用@Configuration）
@EnableAutoConfiguration
@ComponentScan

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```
# @Configuration和@bean
提到@Configuration就要提到他的搭档@Bean。使用这两个注解就可以创建一个简单的spring配置类，可以用来替代相应的xml配置文件中配置bean。
**SpringBoot社区推荐使用基于JavaConfig的配置形式**通过@Configuration+@Bean形式实现
**@Configuration的注解类标识这个类可以使用Spring IoC容器作为bean定义的来源。**
**@Bean注解告诉Spring，一个带有@Bean的注解方法将返回一个对象，该对象应该被注册为在Spring应用程序上下文中的bean。**

```java
xml形式的IOC容器配置类
xml形式IOC容器中配置bean
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
       default-lazy-init="true">
    <!--bean定义-->
    <bean id="mockService" class="..MockServiceImpl">
</bean>
</beans>

JavaConfig形式的IOC容器的配置类
IOC容器中配置bean
@Configuration
public class MockConfiguration{
    //bean定义
     @Bean
    public MockService mockService(){
        return new MockServiceImpl();
    }
}

```
# @ComponentScan
@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。
basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，**如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。**

> 这就是为什么启动类放在root packages下，因为默认扫描的是加了@ComponentScan所在类的package进行扫描，这样启动类放在包的根路径下，扫描到的就是整个项目的bean

# @EnableAutoConfiguration
**@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。**
总结：@EnableAutoConfiguration核心是用到了@Import(EnableAutoConfigurationImportSelector.class)，其中EnableAutoConfigurationImportSelector中用到了SpringFactoriesLoader主要功能是从指定的配置文件META-INF/spring.factories加载配置，


@EnableAutoConfiguration核心就是借助
@Import(EnableAutoConfigurationImportSelector.class)
EnableAutoConfigurationImportSelector核心是用到了SpringFactoriesLoader其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。
完成从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。
**最终会根据类路径中的jar依赖为项目进行自动配置，如：添加了spring-boot-starter-web依赖，会自动添加Tomcat和Spring MVC的依赖，Spring Boot会对Tomcat和Spring MVC进行自动配置。**
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

# SpringBoot Starter是什么？
SpringBoot还有一大优点就是把我们常用的场景抽取成了一个个stater场景启动器，我们通过引入Springboot官方为我们提供的场景启动器然后加上少量的配置就可以实现相应的功能，例如spring-boot-starter-web包自动帮我们引入了web模块开发需要的相关依赖，这些场景启动器starter会把所有用到的依赖都给包含进来，避免了开发者自己去引入依赖所带来的麻烦。

# SpringBoot优点
1. 独立运行的Spring项目：Springboot可以以jar包的形式独立运行项目
2. 内嵌Servlet容器：SpringBoot可以选择内嵌Tomcat jetty容器
3. 官方提供了50多个场景启动器Starter：简化maven的配置
4. 自动化配置Spring，约定大于配置做到真正的0配置
