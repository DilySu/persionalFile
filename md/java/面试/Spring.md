## 1、Spring的理解

Spring 是一个以 **IOC** 和 **AOP** 为核心的**松耦合**的**非侵入式**的**分层式**企业级应用开发框架

> 非侵入式：可以在对现有结构不影响的前提下增强 JavaBean 的功能（DI）
>
> 松耦合：AOP、DI

## 2、Spring IOC 的理解

Spring 中 IOC 就是将对象的创建交给 Spring 容器来完成，再通过依赖注入来使用容器中的对象。

## 3、Spring AOP 的理解

Spring AOP 面向切面编程，就是将区别于业务之外的功能独立出来，进行管理和扩展。

Spring AOP 是基于 IOC 实现的，本质实在 Bean 初始化过程中 通过 Bean后置处理器 中的动态代理来做扩展。

> 关键字： 切面、切点、通知（before、after、around、after throwing、after returing）

## 4、Spring AOP 的应用有哪些？

日志、动态代理、Bean的增强处理器、事务管理

## 5、Bean 的生命周期

Bean 的定义信息 => Bean 的实例化 => Bean 的初始化 => Bean 的使用 => Bean 的销毁 

## 6、Spring是如何解决循环依赖的？

使用三级缓存，将Bean在初始化完成之前进行提前暴露

## 7、Spring 中反射的标准代码流程

1. 获取 Class 对象
2. 获取 Class 对象的构造器
3. 创建对象

## 8、依赖注入的方式有哪些？

注解注入、构造器注入、Setter 注入

## 9、Spring 中 声明 Bean 的注解有哪些？

- Spring 自带

  @Bean：作用于方法，一般用于配置类中

  @Component、@Service、@Controller、@Repository、@RestController：作用于类

- mybatis plus

  @Mapper：作用于类

## 10、Spring 中事务的传播性？

事务的传播性产生于事务的嵌套，内层事务不会对外层造成影响，所以事务的传播性指的其实是外部事务对内部事务的影响，所以对于内部事务，主要分为三大类：

- **死活不能有事务的**：

  Never：外部有事务就抛出异常

  Not_Support: 将外部事务挂起后非事务执行

- **必须要有事务的**：

  Required：外部有就用外部事物，没有就创建事务(SpringBoot @Transactional 注解默认是这个)

  Required_new: 不管外部有没有事务，都创建自己的事务，外部有事务就挂起

  Nested：外部没有事务就创建事务，外部有事务就创建事务并嵌套在外部事务中

  Mandatory：外部没有事务就报异常，外部有事务就用外部事务

- **可有可无的**：

  Supported: 外部有事务就用外部事务，没有就算了

## 11、事务的隔离级别以及可能出现的问题

隔离级别：read uncommited < read committed < repeatable read < seriazable

可能的问题：脏读、幻读、不可重复读、回滚覆盖、提交覆盖

数据库：MySQL默认：repeatable read

​			  SqlServer、Oracle：read committed

