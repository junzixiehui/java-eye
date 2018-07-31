
- 学习spring步骤
- 学习那些东西
- 设计模式
- 底层原理
- 借鉴的地方
- 


#### 学习内容


- 介绍Spring IoC, AOP

- 第二周：Basic BeanFactory

最简单的结构，基于XML的BeanFactory

缺省构造函数的Bean

Resource，BeanDefinitionRegistry 的抽象

Excpeiton的处理

BeanDefinition接口

单一职责的应用

- 第三周：实现setter注入

Bean的scope问题，SingletonBeanRegistry接口

PropertyValue，RuntimeBeanReference，BeanDefinitionValueResolver的抽象

使用Introspector

TypeConverter 实现从字符串到特定类型的转换

从createBean到initiateBean和populateBean的拆分。

- 第四周： 实现构造函数注入

引入ConstructorArgument

如何找到合适的构造器： ConstructorResolver

TestCase的整理，引入TestSuite

- 第五、六周：实现注解和auto-scan

注解的讲解

使用ASM读取类的Metadata

读取一个包下所有的class 作为Resource 

新的BeanDefinition实现类引入

给auto-scan的Bean 命名： BeanNameGenerator

新的抽象： DependencyDescriptor，InjectionMetadata ，InjectedElement

用AutowiredAnnotationProcessor实现注入

Bean的生命周期

BeanPostProcessor接口及其实现和调用

- 第七、八、九周：实现AOP

准备工作：讲解CGLib和Java动态代理的原理，讲解PointCut， Advice....等AOP概念

实现Pointcut 和 MethodMatcher，MethodLocatingFactory

给定一个对象及相应的方法和一系列拦截器(beforeAdvice,afterAdvice)，需要实现正确的调用次序

实现CGLibProxyFactory

实现“合成”Bean

实现对resolveInnerBean

使用AspectJAutoProxyCreator 进行封装

用Java 动态代理实现AOP



- https://github.com/seaswalker/Spring