# 面试题
## 1. Spring Boot 的自动装配怎么实现？
通过@SpringBootApplication中的@EnableAutoConfiguration的@Import从autoconfiguration包中META-INFO/spring-factories文件中获取所有
spring需要装载的bean，同时它还会从整个项目的依赖的jar中的META-INFO/spring-factories文件中扫描需要装载的bean，这也是中间件整合进springboot
的方式。

## 2. Spring 的事务如何实现的挂起？
Spring 的事务挂起依靠的是 ThreadLocal 来实现的。事务挂起时会清空 ThreadLocal 中与当前线程有关的数据，包括连接、事务隔离级别、事务传播等级，
并且将清空的数据返回，由当前事务持有。

参考资料：https://my.oschina.net/anur/blog/3155627 https://blog.csdn.net/qqqqq1993qqqqq/article/details/77991338

## 3. Spring 是如何解决循环依赖的？
Spring 的对象装配的过程其实是一个递归调用的过程，AbstractBeanFactory 的 doGetBean 是整个 Spring 对象装配的核心方法，这个方法里有个 getSingleton
方法，首先判断了 singletonObjects 容器中是否有这个 Bean，没有的话，判断 earlySingletonObjects 中是否有这个 bean，如果也没有就去看 singletonFactories
中有没有这个 bean，如果有的话返回。singletonFactories 里存放的是半成品的bean，就是对象创建完成，但是对象的属性没有被赋值。第一次调用 getSingleton
后，如果没有返回 bean，那么会走到 createBean 的代码逻辑中，createBean 中会首先反射构造方法生成对象实例，随后会将 bean 的引用放到 singletonFactories
中。
参考资料：https://zhuanlan.zhihu.com/p/84267654

## 4. BeanFactory 和 FactoryBean 有什么区别？
BeanFactory 是底层的IOC容器，FactoryBean 是创建 Bean 的一种方式，为了解决复杂对象初始化的场景。

参考资料：https://time.geekbang.org/course/detail/100042601-187470

## 5. 什么情况下事务注解会失效？




