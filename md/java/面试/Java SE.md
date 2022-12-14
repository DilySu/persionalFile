## 1、HashMap 扩容对性能有何影响？

每次扩容都需要对旧的数据重新计算 hash 值，影响性能，所以如果预期知道目标集合大小的话，直接初始化一个比较大的map，一定程度可以提高性能。

HashMap 默认容量达到 0.75（装载因子）时进行扩容。

## 2、HashMap 底层结构是怎样的？

HashMap 底层使用 数组 + 单链表 实现的

## 3、java 中线程创建的方式有哪些？

- 继承 Thread 重写 run 方法
- 实现 runnable 接口，重写 run 方法
- 实现 callable 接口，重写 call 方法，使用线程池提交任务

## 4、线程安全的集合有哪些？

- **HashTable**

  使用 synchronized 对整个对象加锁，性能消耗很大

- **Vector**

  使用 synchronized 对整个对象加锁，性能消耗很大

- **ConcurrentXXX**

  使用 synchronized 对单节点加锁，性能消耗小

## 5、线程的生命周期

线程新建、线程就绪、线程堵塞、线程运行、线程死亡

## 6、线程中 sleep 和 wait 的区别？

wait：释放对象的锁，进入阻塞状态，被唤醒后进行竞争，竞争成功后进入运行状态

sleep：可以模拟网络延迟，但是不会释放对象的锁

## 7、线程同步与线程通信之间的区别？

线程同步：线程同步是不同线程的共享资源的同步

线程通信：线程通信是不同线程之间业务的交流

## 8、== 和 equals 的区别

== ：基本数据类型用于比较值，引用类型用于比较内存地址

equals：引用类型若不重写该方法则也是比较内存地址，一般重写后用于比较内容

## 9、JDK 1.8新特性有哪些？

- Lambda表达式
- stream API
- 函数式接口
- LocalDate、LocalTime、LocalDateTime新时间
- Optional 工具类
- 方法引用和构造引用
- 接口中默认方法和静态方法

## 10、接口和抽象类的区别

语法层次：

- 接口多继承、抽象类单继承
- 接口实现，抽象类继承
- 接口不能有方法体，抽象类有方法体
- 接口定义变量只能是静态常量，抽象类则是普通变量

思想层面：

- 接口：自上向下

  不需要考虑子类实现

- 抽象类：自下向上

  考虑子类共同特点，抽取共同特点生成抽象类

语义层面

- 接口：主要用于描述特征
- 抽象类：主要用于描述概念
