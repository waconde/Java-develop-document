# Spring 知识点

## Spring 定义
轻量级的开源的 J2EE 框架。它是一个容器框架，用来装 javabean (java 对象)的中间层框架(万能胶)。
可以起一个连接作用，比如说把 struts 和 hibernate 粘合在一起运用，可以让我们的企业开发更快、更简洁。  
Spring 是一个轻量级的控制反转(loc)和面向切面(AOP)的容器框架  
-从大小与开销两方面来讲，Spring 都是轻量级的。  
-通过控制反转(loC)的技术达到松耦合的目的  
-提供了面向切面编程的丰富支持，允许通过分离应用的业务逻辑与系统级服务进行内聚性的开发  
-包含并管理应用对象(Bean)的配置和生命周期，这个意义上是一个容器。  
-将简单的组件配置、组合成为复杂的应用，这个意义上是一个框架。

## Spring、Spring MVC、Spring Boot
Spring 是一个 IOC 容器，用来管理 Bean。使用依赖注入实现控制反转，可以很方便的整合各种框架。
提供 AOP 机制弥补 OOP 的代码重复问题、更方便将不同类不同方法中的共同处理抽取成切面、自动注入给方法执行。比如日志、异常等。  
Spring MVC 是 Spring 对 web 框架的一个解决方案，提供了一个总的前端控制器 Servlet，用来接收请求，
然后定义了一套路由策略(url 到 handle 的映射)及适配执行 handle，将 handle 结果使用视图解析技术生成视图展现给前端。  
Spring Boot 是 Spring 提供的一个快速开发工具包，让程序员能更方便、更快速地开发 Spring + Spring MVC 应用，简化了配置(约定大于配置)。
整合了一系列的解决方案(starter 机制)，redis、mongodb、es，可以开箱即用。

## Spring MVC 工作流程
1. 用户发送请求至前端控制器 DispatcherServlet
2. DispatcherServlet 收到请求调用 HandlerMapping 处理器映射器（就是 Url 和 Handler 的映射，Map<Url, Handler>）
3. 处理器映射器找到具体的处理器(可以根据 xml 配置、注解进行查找)，生成处理器及处理器拦截器(如果有则生成)一并返回给 DispatcherServlet
4. DispatcherServlet 调用 HandlerAdapter 处理器适配器
5. HandlerAdapter 经过适配调用具体的处理器(Controller，也叫后端控制器)
6. Controller 执行完成返回 ModelAndView
7. HandlerAdapter 将 Controller 执行结果 ModelAndView 返回给 DispatcherServlet
8. DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器
9. ViewReslover 解析后返回具体View
10. DispatcherServlet 根据 View 进行渲染视图(即将模型数据填充至视图中)
11. DispatcherServlet 响应用户

## Spring Controller 保证线程安全
Controller 是单例模式，但因为 Controller 类，以及其引用的 Service 类、Mapper 类均为无状态的，所以不存在线程安全问题。
所以要注意，在这些类中不要携带数据。

## Spring 的整体框架
1、 WEB:  Web、Servlet、Portlet、Web Socket  
2、 DATA ACCESS: JDBC、ORM、OXM、JMS、Transaction  
3、 Core container： Beans、Core、Context、SpEL  
4、 AOP、AspectJ、Intrumentions、Messaging  
5、 Test

## Bean的生命周期
1. 实例化：创建一个 Bean 对象，new 一个对象或者创建一个动态代理（AOP）
（时机：1、当客户端向容器申请一个 Bean 时；2、当容器在初始化一个 Bean 时发现还依赖了其他 Bean），
Spring 扫描该 Bean 的定义时，会生成 BeanDefinition 对象
2. 设置对象属性（依赖注入），Spring 检查 BeanDefinition 对象，将其他 Bean 注入该 Bean。
3. 检查 Aware 相关接口，如果实现了则调用相应的方法
 >  BeanNameAware:   
 >  BeanClassLoaderAware:   
 >  BeanFactoryAware:  
 >  ApplicationContextAware: 
4. 前置处理钩子，执行 BeanPostProcess 的 postProcessBeforeInitialization()
5. Spring 会检测 Bean 是否实现 InitializingBean 接口，有实现则调用 afterPropertiesSet()
6. init 初始化：如果 Spring 发现 Bean 的方法上配置了 @PostConstruct 注解，或者 xml 中配置了 init-method，则调用该方法
7. 后置处理钩子，执行 BeanPostProcess 的 postProcessAfterInitialization()
8. 销毁前，当 Spring 发现 Bean 实现了 DisposableBean 接口，则会调用 destroy()
9. 销毁：如果 Spring 发现 Bean 的方法上配置了 @PreDestroy 注解，或者 xml 中配置了 destroy-method，则调用该方法

## Spring事务的传播级别
用于 @Transaction 注解中
1. REQUIRED： 如果存在事务，则加入当前事务，如果没有，创建一个事务并加入当前事务。 
2. SUPPORT： 如果存在事务，则加入当前事务，如果没有，不使用。 
3. MANDATORY: 如果存在事务，则加入当前事务，如果不存在抛出异常。 
4. REQUIRED_NEW: 如果有事务存在，挂起当前事务，创建一个新的事务，且不加入当前事务。
5. NOT_SUPPORT：以非事务的方式运行，如果有事务，挂起当前事务。 
6. NEVER： 如果有事务，报错。
7. NESTED： 如果有事务，嵌套事务执行（新建一个事务，并嵌套到当前事务中），如果没有则创建新事务并加入当前事务。 

NESTED、REQUIRED_NEW 和 REQUIRED 的区别：  
对于 REQUIRED_NEW，A 方法有事务，调用存在事务的 B 方法时，那么 A 方法的事务将被挂起，B 方法开启一个自己的事务。  
即：A 和 B 互不影响，是完全独立的两个事务。回滚行为互相独立。（不相关）  
对于 NESTED，A 方法有事务，调用存在事务的 B 方法时，那么 B 方法的事务会嵌套到 A 方法的十五中。  
即：A 为父事务，B 为子事务，父事务回滚，则子事务一起回滚。而子事务回滚，父事务不一定回滚，可判断子事务是否失败进行其他处理。（弱相关）  
对于 REQUIRED，A 方法有事务，调用存在事务的 B 方法时，那么 B 方法的事务会加入到 A 方法的事务中。  
即：它们可视为一个事务，A 或 B 任一个回滚，都会导致 A 和 B 整体回滚。（强相关）  

## SpringBoot 自动装配
自动配置类由各个 starter 提供，它们会使用 @Configuration + @Bean 定义自己的配置类，
并放到 META-INF/spring.factories 下，而后使用 Spring spi 扫描 META-INF/spring.factories 下的配置类。
最后使用 @Import 导入自动配置类
![SpringBoot自动装配.png](../图片素材/Spring/SpringBoot自动装配.png)
