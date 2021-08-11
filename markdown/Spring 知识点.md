# Spring 知识点

## Spring 的整体框架
1、 WEB:  Web、Servlet、Portlet、Web Socket  
2、 DATA ACCESS: JDBC、ORM、OXM、JMS、Transaction  
3、 Core container： Beans、Core、Context、SpEL  
4、 AOP、AspectJ、Intrumentions、Messaging  
5、 Test


## Bean的生命周期

1.  创建一个Bean对象  
2. 检查Aware相关接口    
 >  BeanNameAware:   
 >  BeanFactoryAware:  
 >  ApplicationContentAware: 

3. 执行BeanPostProcess的postProcessBeforeInitializition()
4. init初始化 
5. 执行BeanPostProcess的postProcessAfterInitializition()
6. 销毁 destory-method

## Spring事务的传播级别
1. REQUIRED： 支持当前事务，如果没有，创建一个。 
2. SUPPORT： 支持当前事务，如果没有，不使用。 
3. MANDATORY: 支持当前事务，如果不存在抛出异常。 
4. REQUIRED_NEW: 如果有事务存在，挂起当前事务，创建一个新的。
5. NOT_SUPPORT：以非事务的方式运行，如果有事务，挂起当前事务。 
6. NEVER： 如果有事务，报错。
7. NESTED： 如果有事务，嵌套事务执行。 

## SpringBoot自动装配
