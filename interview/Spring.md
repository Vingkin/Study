## 0x00. 列举一些重要的Spring模块

下图对应的是 Spring4.x 版本。目前最新的 5.x 版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 WebFlux 组件。

![](https://vingkin-1304361015.cos.ap-shanghai.myqcloud.com/interview/e0c60b4606711fc4a0b6faf03230247a.png)

1. Spring Core：核心模块，Spring其他功能基本都需要依赖于该类库，主要提供IOC依赖注入功能的支持
2. Spring Aspects：该模块为与AspectJ的集成提供支持
3. Spring AOP：提供了面向切面编程的实现
4. Spring Data Access / Integration
   1. spring-jdbc : 提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响。
   2. spring-tx : 提供对事务的支持。
   3. spring-orm : 提供对 Hibernate 等 ORM 框架的支持。
   4. spring-oxm ： 提供对 Castor 等 OXM 框架的支持。
   5. spring-jms : Java 消息服务。
5. Spring Web
   1. spring-web ：对 Web 功能的实现提供一些最基础的支持。
   2. spring-webmvc ： 提供对 Spring MVC 的实现。
   3. spring-websocket ： 提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
   4. spring-webflux ：提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步.
6. Spring test：Spring 团队提倡测试驱动开发（TDD）。有了控制反转 (IoC)的帮助，单元测试和集成测试变得更简单。

## 0x01. 请你说说Spring的核心是什么？

Spring的核心是IoC和AOP

1. `IoC`叫反转控制，就是将对象的控制权交由Spring框架来管理。IOC可以帮助我们维护对象与对象之间的依赖关系，降低对象之间的耦合度。

   说到IoC就不得不说DI，IoC是通过DI来实现的。由于IoC这个词汇比较抽象而DI却更直观，所以很多时候我们就用DI来替代它，在很多时候我们简单地将IoC和DI划等号，这是一种习惯。而实现依赖注入的关键是IoC容器，它的本质就是一个工厂。

   DI主要有两种注入方式：

   1. 构造方法注入
   2. setter方法注入

2. `AOP`是面向切面编程的意思。将那些与业务无关却为业务模块共同调用的逻辑或责任（如事务处理、日志管理和权限管理等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，也有利于未来的可拓展性和可维护性。如果@Transactional注解就是通过AOP实现的。

   Spring AOP是基于动态代理的。如果代理对象实现了某个接口，那么Spring AOP会使用JDK Proxy去创建代理对象，对于没有实现接口的对象，Spring AOP会使用Cglib生成一个被代理对象的子类来作为代理。

   当然也可以使用AspectJ！

## 0x02. Spring AOP和AspectJ AOPO的区别？

1. Spring AOP输入运行时增强，而AspectJ是编译时增强。
2. Spring AOP基于代理，而AspectJ基于字节码操作。

## 0x03. 说一说对Spring容器的了解

Spring主要提供了两种类型的容器：BeanFactory和ApplicationContext

* `BeanFactory`：是基础类型的IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略。只有当客户端对象需要访问容器中的某个对象时，该对象才会进行初始化以及依赖注入的操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的IoC容器选择。
* `ApplicationContext`：他是在BeanFactory的基础上构建的，拥有BeanFactory的所有支持。除此之外，还支持比如事件发布、国际化信息支持等。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并进行依赖注入。所以，对于BeanFactory而言，ApplicationContext要求更多的系统资源。同时，因为在启动时就完成所有初始化，容器启动的时间较BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。

## 0x04. 说一说对BeanFactory的了解

BeanFactory是一个类工厂，与传统类工厂不同的是，BeanFactory是类的通用工厂，可以创建并管理各种类的对象。这些被创建和管理的对象叫做Bean。

BeanFactory是Spring容器的顶层接口，Spring为BeanFactory提供了很多实现，比较常用的比如`AnnotationConfigApplicationContext`，常用的方法比如`getBean()`获取指定名称的Bean。

## 0x05. Spring是如何管理Bean的

Spring通过IoC来管理Bean，我们可以通过XML配置或者注解来进行配置。

以下是管理Bean时常用的一些注解：

1. `@ComponentScan`：用于声明扫描策略。通过它的声明，Spring就知道要哪些包下带声明的类需要被扫描。
2. `@Component`，`@Repository`，`@Service`，`@Controller`用于类上声明Bean，他们的作用一样，只是语义不同。`@Component`用于声明通用的Bean，`@Repository`用于声明DAO层的Bean，`@Service`用于声明业务层的Bean，`@Controller`用于声明视图层的控制器Bean，被这些注解声明的类当被扫描到时就会创建对应的Bean
3. `@Autowired`，`@Qualifier`，`@Resource`，`@Value`用于注入Bean。`@Autowired`用于按类型注入，`@Qualifier`指定Bean名称注入需要与`@Autowired`一起使用，`@Resource`既可以按类型注入也可以指定Bean名称注入，@Value适用于注入基本类型。
4. `@Scope`用于声明Bean的作用域。
5. `@PostConstruct`，`@PreDestory`用于声明Bean的生命周期。其中被`@PostConstruct`修饰的发给发将在Bean 实例化后被调用，`@PreDestory`修饰的方法将在容器销毁前调用。

## 0x06. Bean的作用域

默认情况下，Bean在Spring容器中时单例的，可以通过@Scope注解修改Bean的作用域。

| 类型          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| singleton     | 单例Bean，默认                                               |
| prototype     | 每次请求都会创建一个新的bean实例                             |
| request       | 每一次HTTP请求都会产生一个新的Bean                           |
| session       | 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean |
| globalSession | 同一个全局的Session共享一个Bean，一般用于Portlet环境         |

## 0x07. 单例Bean的线程安全问题了解吗

单例Bean存在线程安全问题，主要是因为当多个线程操作同一个对象的时候存在共享资源竞争的问题。

两种解决方法：

1. 在Bean中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中

不过，大部分Bean实际都是无状态（没有实例变量）的（比如Dao、Service），这种情况下，Bean是线程安全的。

