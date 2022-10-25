# 谈谈Spring IOC得理解，原理与实现

**总**：

**控制反转**：理论思想，原来的对象是由使用者来进行控制，有了spring之后，可以把整个对象交给spring来帮我们进行管理

​				**DI**：依赖注入，把对应的属性的值注入到具体的对象中，@Autowired，populateBean完成属性值的注入

**容器**：存储对象，使用map结构来存储，在spring中一般存在三级缓存，singletonObjeacts存放完整的bean对象，整个bean的生命周期，从创建到使用到销毁的过程全部都是由容器来管理的(bean的生命周期)

**分**：

1.一般聊ioc容器的时候涉及到容器的创建过程(beanFactory,DefaultListableBeanFactory),想bean工厂中设置一些参数(BeanPostProcessor,Aeare接口的子类)等等属性

2.加载解析bean对象，准备要创建的bean对象的定义对象beanDefinition(xml或者注解的解析过程)

3.beanFactoryPostProcessor的处理，此处是扩展点，PlaceHolderConfigurSupport，ConfigurationClassPostProcessor

4.BeanPostProcessor的注册功能，方便后续对bean对象完成具体的扩展功能

5.通过反射的方式讲BeanDefinition对象实例化成具体的bean对象

6.bean对象的初始化过程(填充属性，调用aware子类的方法，调用BeanPostProcessor前置处理方法，调用init-mehtod方法，BeanPostProcessor的后置处理方法)

7.生成完整的bean对象，通过getBean方法可以直接获取

8.销毁过程

没看过源码？

​	具体的细节就不清楚了，但是spring中的bean都是通过反射的方式生成的，同时其中包含了很多的扩展点(对占位符的处理)，比如最常用的对BeanFactory的扩展，对bean的扩展，我们在公司对这方面的使用是比较多的，除此之外，ioc中最核心的也就是填充具体bean的属性，和生命周期(背一下)。