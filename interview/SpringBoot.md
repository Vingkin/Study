## 0x00. 说说你对Spring Boot的理解

从本质上来说，Spring Boot就是Spring。Spring Boot使用“约定大于配置”的理念让你的项目快速的运行起来，使用Spring Boot很容易创建一个能独立运行、准生产级别、基于Spring框架的项目，使用Spring Boot你可以不用或者只需要很少的Spring配置。

简而言之，Spring Boot本身并不提供Spring的核心功能，而是作为Spring的脚手架框架，以达到快速构建项目。Spring Boot有如下优点：

* 可以快速构建项目
* 可以对主流开发框架的无配置集成
* 项目可独立运行，无需外部依赖Servlet容器
* 提供运行时的应用监控
* 可以极大地提高开发、部署效率
* 可以与云计算天然集成

## 0x01. Spring Boot Starter有什么用

Spring Boot提供众多起步依赖（Starter）降低项目依赖的复杂度。起步以来本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能。

举例来说，你打算做个Web应用程序。与其向项目的构建文件里添加一堆单独的库依赖，还不如声明这是一个Web应用程序来的简单。你只要添加Spring Boot的Web起步依赖就好了。

# 0x02. 介绍Spring Boot的启动流程

首先，Spring Boot项目创建完成会默认生成一个名为\*Application的入口类，我们是通过该类的main方法启动Spring Boot项目的。在main方法中，通过run方法进行\*Application类的初始化和启动。

\*Application调用run方法的大致流程如下图：

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/4ECC3AECD1D8D2B62421E2D3453DC465.jpg)

其中，\*Application在run方法中重点做了以下操作：

* 获取监听器的参数配置
* 打印Banner信息
* 创建并初始化容器
* 监听器发送通知

当然，除了上述核心操作，run方法运行过程中还涉及启动时长统计、异常报告、启动日志、异常处理等辅助操作。

## 0x03. Spring Boot项目是如何导入包的

通过Spring Boot Starter导入包。其他详见0x01.

[](#0x01. Spring Boot Starter有什么用)

## 0x04. Spring Boot自动装配过程

使用Spring Boot时，我们只需引入对应的Starter，Spring Boot启动时便会自动加载相关依赖，配置相应的初始化参数，以最快捷、简单的形式对第三方软件进行集成，这便是Spring Boot的自动配置功能。

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/4C6D51AEA1E10E3717A8BE4AE88B6F79.jpg)

整个自动配置的过程是：Spring Boot通过@EnableAutoConfiguration注解开启自动配置，加载spring.factories中的各种AutoConfiguration类，当某个AutoConfiguration类满足其注解@Conditional指定的生效条件（Starters提供的依赖、配置或Spring容器中是否存在某个Bean等）时，实例化该AutoConfiguration类中定义的Bean，并注入Spring容器，就可以完成依赖框架的自动配置。

## 0x05. 说说你对Spring Boot的注解的了解

**@SpringBootApplication**

这个注解时Spring Boot项目的基石，创建Spring Boot项目之后会默认在主类加上。

我们可以把`@SpringBootApplication`看作是`@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan`注解的集合。

* `@EnableAutoConfiguration`：启用Spring Boot的自动装配机制
* `@ComponentScan`：包路径扫描，扫描被@Component注解修饰的Bean
* `@SpringBootConfiguration`：就是`@Configuration`的不同语义的版本，允许在该类中使用`@Bean`实修方法注册额外的Bean或导入其他配置类

**@Import**

`@EnableAutoConfiguration`的关键功能就是通过`@Import`注解导入的`ImportSelector`来完成的。从源码得知`@Import({AutoConfigurationImportSelector.class})`是`@EnableAutoConfiguration`注解的组成部分，也是自动配置功能的核心实现者。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

**@Conditional**

`@Conditional`注解是由Spring 4.0版本引入的新特性，可根据是否满足指定的条件来决定是否进行Bean的实例化及装配，比如，设定当类路径下包含某个jar包才会对注解的类进行实例化操作。总之，就是根据一些特定条件来控制实例化的行为。

**@Conditional衍生注解**

* `@ConditionalOnBean`：在容器中有指定Bean的时候才会加载
* `@ConditiaonOnMissingBean`：在容器中没有指定Bean的时候才会加载
* 等等