# 谈一下spring IOC的底层实现

底层实现：工作原理，过程，数据结构，流程，设计模式，设计思想

你对他的理解和你了解过的实现过程

反射，工厂，设计模式，关键的几个方法

createBeanFactory , getBean ,doGetBean , createBean , doCreateBean,createBeanInstance(getDeclaredConstructor(),newinstance),populateBean

1.先通过createBeanFactory 创建一个Bean工厂(DefaultListableBeanFactory)

2.开始循环创建对象，因为容器中的bean默认都是单例的，所以优先通过getBean，doGetBean从容器中查找，找不到的话，

3.通过createBean，doCreateBean方法，以反射的方式创建对象，一般情况下使用的是无参的构造器(getDeclaredConstructor(),newinstance)

4.进行对象的属性填充populateBean

5.进行其他的初始化操作(initializingBean)