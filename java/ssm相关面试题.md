# Spring
## 说说你对Spring的理解
spring的核心主要是两个部分IOC（inversion of control）控制反转和AOP（Aspeect Oriented Programming）
**IOC**
IOC：主要是解决传统方式下在应用程序中的组件需要获取资源（目的使用实例对象方法）时，传统的方式是组件主动的从容器中获取所需要的资源（在组件中主动创建对象实例），在这样的模式下开发人员往往需要知道在具体容器中特定资源的获取方式，同时增加了类与类之间的依赖和耦合度。
控制反转的思想：
反转了资源的获取方向——改由Spring管理的容器主动的推送需要的组件，开发人员不需要知道容器是如何创建资源对象的，只需要提供接收资源的方式即可，IOC容器利用工厂模式和反射降低类之间的耦合，可以在运行时获取IOC容器中创建好的类的实例对象，而不需组件内部主动创建，改变了获取资源的方式，由主动创建改为被动接受。

**AOP**
AOP:面向切面编程：指在程序运行期间，将某段代码动态的切入（不写在业务逻辑方法中）到指定方法的指定位置进行运行的编程方式，主要就是使用jdk动态代理和cglib代理来创建代理对象，通过代理对象来访问目标对象，而代理对象中融入了增强的代码，最终起到对目标对象增强的效果。

## JDK动态代理和Cglib动态代理的区别
JDK动态代理是通过包含对象的实例实现，必须要实现接口
JDK动态代理主要是通过实现InvocationHandler包含被代理对象，并负责分发请求给被代理对象，分发前后均可以做增强。从原理可以看出，JDK动态代理是“对象”的代理。
Cglib动态代理：通过继承对方法	增强，然后利用
**通过“继承”可以继承父类所有的公开方法，然后可以重写这些方法**，**在重写时对这些方法增强**，这就是cglib的思想。根据里氏代换原则（LSP），父类需要出现的地方，子类可以出现。

# SpringBoot
## SpringBoot最大优势
Spring Boot 的最大的优势是“约定优于配置“。“约定优于配置“是一种软件设计范式，开发人员按照约定的方式来进行编程，可以减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。
## Springboot启动工作原理
Springboot的启动原理核心就是两个
一个是@SpringBootApplication注解
另一个SpringApplication.run(Application.class, args);静态方法，run的启动过程比较复杂需要在程序运行中打断点一点点的分析，我这里主要讲讲@SpringBootApplication注解的作用吧
首先@SpringBootApplication注解是一个组合注解点开它我们可以看他用到了多个注解，其中最主要的是三个@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan
1. @SpringBootConfiguration其主要还是用到的@Configuration，@Configuration+@Bean注解其主要作用是实现SpringBoot社区推荐的JavaConfig形式配置Bean其等价于在Spring中是基于xml形式配置bean。
2. @ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。其默认扫描的范围是声明该注解所在类的包下进行扫描，这也解释了为什么启动类必须要包的根路径下。
3. @EnableAutoConfiguration是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器
@EnableAutoConfiguration核心是用到了@Import(EnableAutoConfigurationImportSelector.class)，其中EnableAutoConfigurationImportSelector（导入选择器）中用到了`SpringFactoriesLoader`主要功能是从指定的配置文件META-INF/spring.factories加载配置。
**Spring框架提供的各种名字为@Enable开头的Annotation定义，其实一脉相承：借助@Import的支持，收集和注册特定场景相关的bean定义。**
@EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。


## SpringBoot常用注解
SpringBoot常用注解分为
**- 启动注解**
@SpringBootApplication是一个复合注解主要用到了
@SpringBootConfiguration 注解，继承@Configuration注解，主要用于加载配置文件
@EnableAutoConfiguration 注解，开启自动配置功能
@ComponentScan 注解，主要用于组件扫描和自动装配

**- Controller相关注解**
@Controller：用于处理http请求

@RestController 复合注解=@ResponseBody+@Controller合在一起的作用,RestController使用的效果是将方法返回的对象直接在浏览器上展示成json格式.

@RequestBody：通常是post方法通过HttpMessageConverter读取Request Body并反序列化Object（泛指）对象

@RequestMapping 是 Spring Web 应用程序中最常被用到的注解之一。这个注解会将 HTTP 请求映射到 MVC 和 REST 控制器的处理方法上

@GetMapping用于将HTTP get请求映射到特定处理程序的方法注解

@PostMapping用于将HTTP post请求映射到特定处理程序的方法注解

**- 获取请求参数相关注解**
@PathVariable:获取url中的数据

@RequestPara@RequestHeader 把Request请求header部分的值绑定到方法的参数上m:获取请求参数的值

@CookieValue 把Request header中关于cookie的值绑定到方法的参数上

**- 注入bean相关注解**
@Repository：标识dao层组件

@Service ：标识服务层组件

@Scope作用域注解
作用在类上和方法上，用来配置 spring bean 的作用域，它标识 bean 的作用域
value 在容器中的作用域单例还是多例，在http请求中的作用域
    singleton   表示该bean是单例的。(默认)
    prototype   表示该bean是多例的，即每次使用该bean时都会新建一个对象。
    request     在一次http请求中，一个bean对应一个实例。
    session     在一个httpSession中，一个bean对应一个实例。

@Autowired 自动导入
@Autowired注解作用在构造函数、方法、方法参数、类字段以及注解上
@Autowired注解可以实现Bean的自动注入
 
 @Component
把普通pojo实例化到spring容器中，相当于配置文件中的

**- 导入配置文件相关注解**
@PropertySource注解
引入单个properties文件：
@PropertySource(value = {"classpath : xxxx/xxx.properties"})
引入多个properties文件：
@PropertySource(value = {"classpath : xxxx/xxx.properties"，"classpath : xxxx.properties"})

@ImportResource导入xml配置文件
可以额外分为两种模式 相对路径classpath，绝对路径（真实路径）file
注意：单文件可以不写value或locations，value和locations都可用
相对路径（classpath）
引入单个xml配置文件：@ImportSource("classpath : xxx/xxxx.xml")
引入多个xml配置文件：@ImportSource(locations={"classpath : xxxx.xml" , "classpath : yyyy.xml"})
绝对路径（file）
引入单个xml配置文件：@ImportSource(locations= {"file : d:/hellxz/dubbo.xml"})
引入多个xml配置文件：@ImportSource(locations= {"file : d:/hellxz/application.xml" , "file : d:/hellxz/dubbo.xml"})
取值：使用@Value注解取配置文件中的值
@Value("${properties中的键}")
private String xxx;

Import 导入额外的配置信息
功能类似XML配置的，用来导入配置类，可以导入带有@Configuration注解的配置类或实现了

- 全局异常处理相关注解
@ControllerAdvice 注解定义全局异常处理类
@ExceptionHandler 注解声明异常处理方法
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseBody
    String handleException(){
        return "Exception Deal!";
    }
}
```

# SpringBoot与其他框架的区别

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

# SpringMVC
## 请求处理流程

# JDBC
JDBC的全称是Java DataBase Connection，也就是Java数据库连接，我们可以用它来操作关系型数据库。用JDBC来连接数据库，执行SQL查询，存储等过程，并处理返回的结果。

## JDBC操作数据库的步骤？
1. 注册数据库驱动。
2. 建立数据库连接Connection
3. 创建一个Statement。
4. 执行SQL语句。
5. 处理结果集 ResultSet
6. 关闭数据库相关连接 （ResultSet，statement，Connection）


## JDBC是如何实现Java程序和JDBC驱动的松耦合的？
java通过加载数据库驱动，调用JDBC接口来完成与数据库的相关操作的，所有与数据库的操作都是调用数据库接口,而驱动只有在通过Class.forName反射机制来加载的时候才会出现。

## jdbc,mybatis,hibernate各自优缺点及区别
1）从层次上看，JDBC是较底层的持久层操作方式，**而Hibernate和MyBatis都是在JDBC的基础上进行了封装使其更加方便程序员对持久层的操作。**
2）从功能上看，JDBC就是简单的建立数据库连接，然后创建statement，将sql语句传给statement去执行，如果是有返回结果的查询语句，会将查询结果放到ResultSet对象中，通过对ResultSet对象的遍历操作来获取数据；Hibernate是将数据库中的数据表映射为持久层的Java对象，对sql语句进行修改和优化比较困难；MyBatis是将sql语句中的输入参数和输出参数映射为java对象，sql修改和优化比较方便.
3）从使用上看，如果进行底层编程，而且对性能要求极高的话，应该采用JDBC的方式；如果要对数据库进行完整性控制的话建议使用Hibernate；如果要灵活使用sql语句的话建议采用MyBatis框架。
