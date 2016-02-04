---
layout: default
title: Spring IoC 容器的配置与使用
---

[TOC]

Spring框架很庞大，但IoC容器是核心。IoC是用来管理Spring Bean的，学习IoC其实就是学习Spring Bean的定义和使用。

# 1. 定义Bean
xml是最基本的定义方式，即使是只用标注不用xml也可以通过xml加以说明。Bean可以定义在多个xml里，理论上可以把xml放在任何一个应用能访问到的地方，比如放在一个远程主机，通过http访问它。不过我相信99%的人不会这么干，一般都是放在classpath下的某个容易找到的地方，或者放在本地文件系统中。

定义Bean可以通过xml也可以用标注，实际开发中一般xml和标注都会用，它们是互补的，有各自的优势和应用场景。

例如，使用标注`@Component`、`@Repository`、`@Service`、`@Controller`可以为持久层、业务层、表现层定义Bean，而这是xml做不到的。使用标注也可以让Spring兼容JavaBean和CDI。

一般来说，标注式的Bean定义是硬编码到代码中的，它不能在打包后、部署之前随意更改。xml则相反，它其实是文本，可以随时修改。所以一般建议把一些会根据环境而变化的信息使用xml的方式配置，相当于应用的配置文件。这也是Spring Framework相当便利的地方。而一些绝不能随意更改的信息则使用标注的方式。

## 1.1 XML方式
最基本的xml文件是这样的：

```xml

<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                                http://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="..." class="...">
    <!-- bean相关配置 -->
</bean>

<bean id="..." class="...">
     <!-- bean相关配置 -->
</bean>

    <!-- 更多bean的定义 -->

</beans>
```

根元素是`<beans />`，包含一系列bean元素，它的作用是定义 Spring Bean，这是最常见的。还会有其它的元素实现其它的功能。

### 1.1.1 bean元素及其子元素
bean 元素是 IoC 配置最复杂的元素，基本上 xml 配置的内容分为两大类，bean 元素是一类，其它的统统是一类。从官方文档上就能看出来，bean 元素也是用一个单独的章节加以说明。可见其复杂程度，官方文档又没有集中的列出它的属性和子元素，所以很容易看到后面就忘了前面。写这篇文章的目的其实就是总结 bean 元素的使用。

定义一个 bean 一般需要包含以下几类数据：
- **全限定类名。** 注意这必须是真正的实现类的类名，你绝不能指定一个接口。
- **Bean 行为的配置元素。** 用于描述 Bean 在容器中的行为，如范围、生命周期回调以及其它。
- 对其它 bean 的引用，这种引用也称之为 **协作** 或者 **依赖** 。
- 其它关于对象创建的配置。例如一个 bean 管理多少个连接的连接池，或者是该池的大小限制。

因此，bean 元素的属性及子元素也分成了几大类，有关于创建的，有关于行为的，有关于依赖的。这里先列张表。

表1：bean 元素属性列表

属性             | 取值                                  | 意义
-------------- | ----------------------------------- | -------------------------------------------------------------------------
id             | 小写开头的字母（加数字）                        | 大名或学名，就是它的名称。调用bean，特别是在别的bean元素中引用它的时候只能用这个名称。
name           | 小写开头的字母（加数字），多组字符以逗号或分号或空格分隔        | bean的小名或昵称，标准的说法叫做别名。可以有多个
class          | 全限定类名                               | 类型，必须是真正的实现类。不能是接口或者枚举等等其它的东西
factory-bean   | 某个bean的`id`或`name`                  | 对包含生成实例方法的bean的引用，与`factory-method`属性一起使用
factory-method | 代表方法名的字符串                           | 指定容器构建实例时要调用的方法，如果与`factory-bean`属性同时使用则是动态方法；与`class`属性同时使用表示静态方法。不能单独使用
scope          | protype,session,application,request | 生命期

表2：bean元素的子元素

在详细说明这些属性及元素的使用之前，有一点需要强调一下：**Bean的创建与Bean的依赖注入是两回事**，虽然DI是靠IoC实现的，其实IoC和DI这是两个概念，混在一起就说不清楚了。Spring官方文档上也从来没有把IoC和DI放在一起说过。

前面谈到，bean的属性与子元素有几大类，分别定义创建、行为、依赖，下面逐个说明。

#### 创建Bean实例
实例化Bean的方式主要是通过bean元素的class属性定义，结合其它不同的属性定义不同的创建方式。可以使用两种途径中的一种来定义创建方式：
- 通常情况下，直接使用`class`属性指定的类名的构造方法创建，这在某种程度上相当于`new`操作。
- 通过指定真实的类和所包含的静态工厂方法来创建。所请求的静态工厂方法返回的对象类型可以与指定的类相同也可以不同。

##### 通过构造器创建实例
当你使用构造器这一途径来创建bean时，所有普通的类都是可用并且与Spring兼容的。这表示这些类不需要实现特定的接口或是编写特殊的方法。但不管怎样，要依赖IoC使用你指定的bean，你可能**需要一个默认的无参构造方法**。

使用默认无参构造方法创建实例的bean定义最简单，只要指定`class`属性就行了，类似这样：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

理论上`id`和`name`属性可以忽略，但如果真不指定就没办法调用这个bean了。

> **内部类名。** 如果你要为一个静态的嵌套类配置bean定义的话，你必须使用双重类名。

> 举个例子，如果在`com.example`包中有一个名叫`Foo`的类，而这个`Foo`类又有一个声明为`static`的内部类`Bar`，那么`class`属性的值就应该是... com.example.Foo$Bar

> 注意字符`$`是用在内部与外部类名之间的分隔符。

##### 通过静态工厂方法构建实例
当需要通过一个静态工厂方法来构建Bean的时候，你可以使用`class`属性指定包含了静态工厂方法的类并用`factory-method`属性指定静态工厂方法名。

通过这种方式定义Bean和使用构造器不同的是：`class`属性并非指定Bean的类型，仅仅是说明这个类中有个工厂方法而已。这个方法并不一定返回`class`所指定的类型的对象。

下面的示例中定义的Bean将通过指定的工厂方法创建。示例中指定的`createInstance()`方法必须是静态方法。

```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```

> 很明显，这相当于在前一节的基础上加了一个`factory-method`属性。但`class`属性所代表的意义完全不同。

按照官方文档的意思，这个设定是为了方便实现单例模式。换而言之，这个静态工厂方法一般情况下是返回一个已经存在的实例，只有在没有实例存在的时候才创建一个新实例。

##### 通过动态工厂方法构建Bean
可以通过某个由容器创建的Bean实例中的非静态方法来构建Bean。要使用这个功能时，省略`class`属性；在`factory-bean`属性中指定当前容器（或父容器）中包含用来构建实例的动态工厂方法的Bean；然后在`factory-method`属性中指定要调用的方法名。例如：

```xml
<!-- 定义包含createClientServiceInstance()方法的工厂bean -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 为bean注入必须的依赖 -->
</bean>

<!-- 这个bean将通过指定的工厂bean创建 -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

如上所示，`factory-bean`属性指定的Bean得另外定义，这个Bean必须包含`factory-method`属性要调用的工厂方法。实际上所谓"工厂方法"在这里其实包括任何非静态并且返回值为非`void`的方法，你甚至可以返回一个`Void`类型的对象，不过我想像不到什么样的应用场景需要这么干，但这么干显然不会导致任何异常发生。

同样的道理，你也可以调用某个Bean的多个方法来生成不同的Bean。

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
   <!-- 为这个locator注入必要的依赖 -->
</bean>

<bean id="clientService"
   factory-bean="serviceLocator"
   factory-method="createClientServiceInstance"/>

<bean id="accountService"
   factory-bean="serviceLocator"
   factory-method="createAccountServiceInstance"/>
```

这种定义方式的意义在于可以通过一个Bean来管理其它Bean的创建以及依赖。

### 1.1.2 其它元素
#### 定义别名：alias
使用alias在bean定义之外的任何地方为这个bean定义别名。定义的方式为：

```xml
<alias name="fromName" alias="toName"/>
```

fromName是bean的名字，可以是`id`也可以是`name`中的一个，toName是要为它添加的别名。

# 2. IoC容器
要使用Spring Bean就必须和IoC容器打交道，访问IoC就必须通过`BeanFactory`接口。可以理解为，`BeanFactory`就是IoC的UI，必须通过它来向IoC发送指令。

但`BeanFactory`并不被官方推荐使用。按官方的说法，这个东西是由于一些历史原因，为了第三方框架而存在的，一般的应用应该使用它的子接口`ApplicationContext`。官方文档上是这样说的："使用`ApplicationContext`，除非你有充足的理由不这样做。

## 使用`AnnotationConfigApplicationContext`实例化Spring容器
