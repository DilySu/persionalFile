# 谈一下spring事务传播？

传播特性有几种？七种

Required，Requireds_new,nested,Support,Not_Support,Never,Mandatory

某一个事务嵌套另外一个事务的时候怎么办？

A方法调用B方法，AB方法都有事务，并且传播特性不同，那么A如果有异常，B怎么办，B如果有异常，A怎么办？

------------------

**总**：事务的传播特性指的是不同方法的嵌套调用过程中，事务该如何进行处理，是同一个事务还是不同的事务，当出现异常的时候会回滚还是提交，两个方法之间相互影响，在日常工作中，使用比较多的是required，Requireds_new

**分**：1.先说事务的不同分类，可以分为三类，支持当前事务，不支持当前事务，嵌套事务

​		2.如果外层方法是required，内层方法是：required，requireds_new,nested

​		3.如果外层方法是requireds_new，内层方法是：required，requireds_new,nested

​		4.如果外层方法是nested，内层方法是：required，requireds_new,nested

核心处理逻辑非常简单：

1.判断内外方法是否是同一事务：

​		是：异常统一在外层方法处理

​		不是：内层方法有可能影响到外层方法，但是外层方法是不会影响内层方法的

​		（大致可以这么理解，但是有个别情况不同，nested）