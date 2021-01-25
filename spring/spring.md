# 1、Spring 概述

## 1.1 什么是 Spring

Spring 是一个轻量级 Java 开发框架，最早由 Rod Johnson 创建，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是一个分层的 JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发 Java 应用程序提供全面的基础架构支持。Spring 负责基础架构，因此 Java 开发者可以专注于应用程序的开发

Spring 最根本的使命是解决企业级应用开发的复杂性，即简化 Java 开发

Spring 可以做很多事情，它为企业级开发提供了丰富的功能，但是这些功能的底层都依赖于它的两个核心特性，也就是依赖注入（dependency injection，DI）和面向切面编程（aspect-oriented programming，AOP）

为了降低 Java 开发的复杂性，Spring 采用了以下4种关键策略：

- 基于 POJO 的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 通过切面和惯例进行声明式编程
- 通过切面和模板减少板式代码

## 1.2 Spring 框架的设计目标、设计理念和核心是什么

**Spring 设计目标**：Spring 为开发者提供一个一站式轻量级应用开发平台

**Spring 设计理念**：在 JavaEE 开发中，支持 POJO 和 JavaBean 开发方式，使应用面向接口开发，充分支持 OO（面向对象）设计方法；Spring 通过 IOC 容器实现对象耦合关系的管理，并实现依赖反转，将对象之间的依赖关系交给 IOC 容器，实现解耦

**Spring 框架的核心**：IOC 容器和 AOP 模块。通过 IOC 容器管理 POJO 对象以及他们之间的耦合关系，通过 AOP 以动态非侵入的方式增强服务

IOC 让相互协作的组件保持松散的耦合，而 AOP 编程允许你把遍布于各层的功能分离出来形成可重用的功能组件

## 1.3 Spring 的优缺点

**优点**

- 方便解耦，简化开发

  Spring 就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给 Spring 管理

- AOP 编程的支持

  Spring 提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能

- 声明式事务的支持

  只需要通过配置就可以完成对事务的管理，而无需手动编程

- 方便程序的测试

  Spring 对 Junit4 支持，可以通过注解方便的测试 Spring 程序

- 方便集成各种优秀框架

  Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、Mybatis等）

- 降低 JavaEE API 的使用难度

  Spring 对 JavaEE 开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低

**缺点**

- Spring明明一个很轻量级的框架，却给人感觉大而全
- Spring依赖反射，反射影响性能
- 使用门槛升高，入门Spring需要较长时间

## 1.4 Spring 有哪些应用场景

应用场景：JavaEE 企业应用开发，包括 SSH、SSM 等

Spring 价值：

- Spring 是非侵入式的框架，目标是使应用程序代码对框架依赖最小化
- Spring 提供一个一致的编程模型，使应用直接使用 POJO 开发，与运行环境隔离开来
- Spring 推动应用设计风格面向对象和面向接口开发转变，提高了代码的重用性和可测试性

## 1.5 Spring 由哪些模块组成

Spring 总共大约有 20 个模块，由 1300 多不同的文件构成。而这些组件被分别整合在 `核心容器(Core Container)`、`AOP(Aspect Oriented Programming)和设备支持(Instrmentation)`、`数据访问与集成(Data Access/Integeration)`、`Web`、`消息(Messaging)`、`Test` 等 6 个模块中。以下是 Spring 5 的模块结构图：

![在这里插入图片描述](md-image/2019102923475419.png)

- Spring Core：提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能
- Spring beans：提供了 `BeanFactory`，是工厂模式的一个经典实现，Spring 将管理对象称为 Bean
- Spring context：构建于 core 封装包基础上的 context 封装包，提供了一种框架式的对象访问方法
- Spring jdbc：提供了一个 JDBC 的抽象层，消除了繁琐的 JDBC 编码和数据库厂商特有的错误代码解析，用于简化 JDBC
- Spring aop：提供了面向切面的编程实现，让你可以自定义拦截器、切点等
- Spring web：提供了针对 web 开发的集成特性，例如文件上传，利用 servlet listeners 进行 IOC 容器初始化和针对 web 的 `ApplicationContext`
- Spring test：主要为测试提供支持的，支持使用 Junit 或 TestNG 对 Spring 组件进行单元测试和集成测试

## 1.6 Spring 框架中都用到了哪些设计模式

1. 工厂模式：`BeanFactory` 就是简单工厂模式的体现，用来创建对象的实例
2. 单例模式：`Bean` 默认为单例模式
3. 代理模式：Spring 的 AOP 功能用到了 JDK 的动态代理 和 cglib 字节码生成技术
4. 模板方法：用来解决代码重复的问题。比如：RestTemplate、JMSTemplate、JpaTemplate
5. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如 Spring 中 listener 的实现——`ApplicationListener`

## 1.7 详细讲解一下核心容器(spring context应用上下文)模块

这是基本的 Spring 模块，提供 Spring 框架的基础功能，`BeanFactory` 是任何以 Spring 为基础的应用的核心。Spring 框架建立在此模块之上，它使 Spring 成为一个容器

`Bean` 工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从真正的应用代码中分离。最常用的就是 `org.springframework.beans.factory.xml.XmlBeanFactory`，它根据 XML 文件中的定义加载 beans。该容器从 XML 文件读取配置元数据并用它去创建一个完全配置的系统或应用

## 1.8 Spring 框架中有哪些不同类型的事件

Spring 提供了以下 5 种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：在调用 `ConfigurableApplicationContext` 接口中的 `refresh()` 方法时被触发
2. 上下文开始事件（ContextStartedEvent）：当容器调用 `ConfigurableApplicationContext` 的 `start()` 方法开始/重新开始容器时触发该事件
3. 上下文停止事件（ContextStoppedEvent）：当容器调用 `ConfigurableApplicationContext` 的 `stop()` 方法停止容器时触发该事件
4. 上下文关闭事件（ContextClosedEvent）：当 `ApplicationContext` 被关闭时触发该事件。容器被关闭时，其管理的所有单例 `Bean` 都被销毁
5. 请求处理事件（RequestHandledEvent）：在 web 应用中，当一个 http 请求（request）结束触发该事件。如果一个 bean 实现了 `ApplicationListener` 接口，当一个 `ApplicationEvent` 被发布之后，bean 会自动被通知

## 1.9 Spring 应用程序有哪些不同组件

Spring 应用一般有以下组件：

- 接口 - 定义功能
- Bean 类 - 它包含属性，setter 和 getter 方法，函数等
- Bean 配置文件 - 包含类的信息以及如何配置它们
- Spring 面向切面编程（AOP） - 提供面向切面编程的功能
- 用户程序 - 它使用接口

## 1.10 使用 Spring 有哪些方式

使用 Spring 有以下方式：

- 作为一个成熟的 Spring Web 应用程序。
- 作为第三方 Web 框架，使用 Spring Frameworks 中间层。
- 作为企业级 Java Bean，它可以包装现有的 POJO（Plain Old Java Objects）。
- 用于远程使用

# 2、控制反转

## 2.1 什么是 Spring IOC 容器

控制反转即 IOC（Inversion of Control），它把传统上由程序代码直接操纵的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转“概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器

Spring IOC 负责创建对象，管理对象（通过依赖注入（DI））、装配对象、配置对象，并且管理这些对象的整个生命周期

## 2.2 控制反转(IOC)有什么作用

管理对象的创建和依赖关系的维护。对象的创建并不是一件简单的事，在对象关系比较复杂时，如果依赖关系需要程序员来维护的话，那是相当头疼的

解耦，由容器去维护具体的对象

托管了类的产生过程，比如我们需要在类的产生过程中做一些处理，最直接的例子就是代理，如果有容器程序可以把这部分处理交给容器，应用程序则无需去关心类是如何完成代理的

## 2.3 IOC 的优点是什么

- IOC 或依赖注入把应用的代码量降到最低
- 它使应用功能容易测试，单元测试不再需要单例和 JNDI 查找机制
- 最小的代价和最小的侵入性使松散耦合得以实现
- IOC 容器支持加载服务时的饿汉式初始化和懒加载

## 2.4 Spring IOC 的实现机制

Spring 中的 IOC 的实现原理就是工厂模式加反射机制

示例：

```java
interface Fruit {
   public abstract void eat();
 }

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f = null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```

## 2.5 Spring 的 IOC 支持哪些功能

Spring 的 IOC 设计支持以下功能：

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调某些方法（但是需要实现 Spring 接口，略有侵入）

其中，最重要的就是依赖注入，从 XML 的配置上说，即 ref 标签，对应 `Spring RuntimeBeanReference` 对象

对于 IOC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入

## 2.6 BeanFactory 和 ApplicationContext 有什么区别

`BeanFactory` 和 `ApplicationContext` 是 Spring 的两大核心接口，都可以当做 Spring 的容器。其中 `ApplicationContext` 是 `BeanFactory` 的子接口

**依赖关系**

`BeanFactory` 是 Spring 里面最底层的接口，包含各种 `Bean` 的定义，读取 bean 配置文档，管理 bean 的加载、实例化，控制 bean 的生命周期，维护 bean 之间的依赖关系

`ApplicationContext` 接口作为 `BeanFactory` 的派生，除了提供 `BeanFactory` 具有的功能外，还提供了更完整的框架功能：

- 继承 `MessageSource`，因此支持国际化
- 统一的资源文件访问方式
- 提供在监听器中注册 bean 的事件
- 同时加载多个配置文件
- 载入多个（有继承关系）上下文，使得每一个上下文都专注于一个特定的层次，比如应用的 web 层

**加载方式**

`BeanFactory` 采用的是延迟加载形式来注入 Bean 的，即只有在使用到某个 Bean 时（调用 `getBean()`），才对该 Bean 进行加载实例化。这样，我们就不能发现一些存在的 Spring 配置问题。如果 bean 的某一个属性没有注入，`BeanFactory` 加载后，直至第一次使用调用 `getBean` 方法才会抛出异常

`Application` 是在容器启动后，一次性创建了所有的 Bean。这样，在容器启动时，我们就可以发现 Spring 中存在的配置错误，这样有利于检查所依赖属性是否注入。`ApplicationContext` 启动后预载入所有的单实例 Bean，通过预载入单实例 bean，确保当你需要的时候，你就不用等待，因为他们已经创建好了

相对于基本的 `BeanFactory`、`ApplicationContext` 唯一的不足就是占用内存空间。当应用程序配置 Bean 较多时，程序启动较慢

**创建方式**

`BeanFactory` 通常以编程的方式被创建，`ApplicationContext` 还能以声明的方式创建，如使用 `ContextLoader`

**注册方式**

`BeanFactory` 和 `ApplicationContext` 都支持 `BeanPostProcessor`、`BeanFactoryPostProcessor` 的使用，但两者之间的区别是：`BeanFactory` 需要手动注册，而 `ApplicationContext` 则是自动注册

## 2.7 Spring 如何设计容器的，BeanFactory 和 ApplicationContext 关系

Spring 作者 Rod Johnson 设计两个接口用以表示容器

- BeanFactory
- ApplicationContext

`BeanFactory` 简单粗暴，可以理解为就是个 `HashMap`，Key 是 `BeanName`，Value 是 `Bean` 实例。通常只提供注册（put），获取（get）这两个功能，我们可以称之为“低级容器”

`ApplicationContext` 可以称之为”高级容器“。因为它比 `BeanFactory` 多了更多的功能。它集成了多个接口。因此具备了更多的功能。例如资源的获取，支持多种消息（例如 JSP tag的支持），对 `BeanFactory` 多了工具级别的支持等等。所以，从名字看，已经不是 `BeanFactory` 之类的工厂了，而是”应用上下文“，代表着整个大容器的所有功能。该接口定义了一个 `refresh` 方法，此方法是所有阅读 Spring 源码的人最熟悉的方法，用于刷新整个容器，即重新加载/刷新所有的 bean

当然，除了这两个大接口，还有其他的辅助接口，这里就不一一介绍了

为了更直观的展示 `BeanFactory` 和 `ApplicationContext` 的关系，这里通过常用的 `ClassPathXMLApplicationContext` 类来展示整个容器的层级 UML 关系

![img](md-image/20191105111441363.png)

有点复杂？ 先不要慌，我来解释一下。

最上面的是 BeanFactory，下面的 3 个绿色的，都是功能扩展接口，这里就不展开讲。

看下面的隶属 ApplicationContext 粉红色的 “高级容器”，依赖着 “低级容器”，这里说的是依赖，不是继承哦。他依赖着 “低级容器” 的 getBean 功能。而高级容器有更多的功能：支持不同的信息源头，可以访问文件资源，支持应用事件（Observer 模式）。

通常用户看到的就是 “高级容器”。 但 BeanFactory 也非常够用啦！

左边灰色区域的是 “低级容器”， 只负载加载 Bean，获取 Bean。容器其他的高级功能是没有的。例如上图画的 refresh 刷新 Bean 工厂所有配置，生命周期事件回调等。

小结

说了这么多，不知道你有没有理解Spring IoC？ 这里小结一下：IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里。
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

## 2.8 ApplicationContext 通常的实现是什么

`FileSystemXmlApplicationContext`：此容器从一个 XML 文件中加载 beans 的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数

`ClassPathXmlApplicationContext`：此容器也从一个 XML 文件中加载 beans 的定义，这里，你需要正确设置 classpath 因为这个容器将在 classpath 里找 bean 配置

`WebXmlApplicationContext`：此容器加载一个 XML 文件，此文件定义了一个 WEB 应用的所有 bean

## 2.9 什么是 Spring 的依赖注入

控制反转 IOC 是一个很大的概念，可以用不同的方式来实现。其主要实现方式有两种：依赖注入和依赖查找

依赖注入：相对于 IOC 而言，依赖注入（DI）更加准确地描述了 IOC 的设计理念。所谓依赖注入（Dependency Injection），即组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联的组件之中。组件不做定位查询，只提供普通的 Java 方法让容器去决定依赖关系

## 2.10 依赖注入的基本原则

依赖注入的基本原则是：应用组件不应该负责查找资源或者其他依赖的协作对象。配置对象的工作应该由 IOC 容器负责，“查找资源“ 的逻辑应该从应用组件的代码中抽取出来，交给 IOC 容器负责。容器全权负责组件的装配，它会把符合依赖关系的对象通过属性（Javabean 中的 setter）或者是构造器传递给需要的对象

## 2.11 有哪些不同类型的依赖注入实现方式

依赖注入时时下最流行的 IOC 实现方式，依赖注入分为接口注入（interface injection），Setter 方法注入（setter injection）和构造器注入（Constructor injection）三种方式。其中接口注入由于在灵活性和易用性比较差，现在从 Spring4 开始已被废弃

## 2.12 构造器依赖注入和 Setter 方法注入的区别

|      **构造函数注入**      |    **setter** **注入**     |
| :------------------------: | :------------------------: |
|        没有部分注入        |         有部分注入         |
|    不会覆盖 setter 属性    |     会覆盖 setter 属性     |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
|     适用于设置很多属性     |     适用于设置少量属性     |

两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖。

# 3、Spring Bean

## 3.1 什么是 Spring Beans

Spring Beans 是那些形成Spring应用的主干的 Java 对象。它们被 Spring IOC 容器初始化，装配，和管理。这些 beans 通过容器中配置的元数据创建。比如，以 XML 文件中的形式定义

## 3.2 一个 Spring Bean 定义包含什么

一个 Spring Bean 的定义包含容器必知的所有配置元数据，包括如何创建一个 bean，它的生命周期详情以及它的依赖

## 3.3 如何给 Spring 容器提供配置元数据？

这里有三种重要的方法给 Spring 容器提供配置元数据

- XML 配置文件
- 基于注解的配置
- 基于 Java 的配置

## 3.4 Spring 配置文件包含了哪些信息

Spring 配置文件是一个 XML 文件，这个文件包含了类信息，描述了如何配置它们，以及如何相互调用

## 3.5 Spring 基于 XML 注入 bean 的几种方式

1. Set 方法注入
2. 构造器注入：① 通过 index 设置参数的位置    ② 通过 type 设置参数类型
3. 静态工厂注入
4. 实例工厂

## 3.6 怎样定义类的作用域

当定义一个在 Spring 里，我们还能给这个 bean 声明一个作用域。它可以通过 bean 定义的 scope 属性来定义。如，当 Spring 要在需要的时候每次生产一个新的 bean 实例，bean 的 scope。另一方面，一个 bean 每次使用的时候必须返回一个实例，这个 bean 的 scope 属性必须设置为 singleton

## 3.7 Spring 支持的几种 bean 的作用域

Spring框架支持以下五种bean的作用域：

- **singleton :** bean在每个Spring ioc 容器中只有一个实例。
- **prototype**：一个bean的定义可以有多个实例。
- **request**：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **session**：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **global-session**：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

注意：缺省（默认）的 Spring bean 的作用域是 singleton。使用 prototype 作用域需要慎重的考虑，因为频繁地创建和销毁 bean 会带来很大的性能开销

## 3.8 Spring 中的单例 bean 是线程安全的吗

不是，Spring 框架中的单例 bean 不是线程安全的

Spring 中的 bean 默认是单例的，Spring 框架并没有对单例 bean 进行多线程的封装处理

实际上大部分时候 Spring bean 无状态的（比如 doa 层），所以某种程度上来说 bean 也是安全的，但如果 bean 有状态的（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把 `singleton` 更改为 `prototype`，这样请求 bean 相当于 new Bean() 了，所以就可以保证线程安全了

- 有状态就是有数据存数功能
- 无状态就是不会保存数据

## 3.9 Spring 如何处理线程并发问题

在一般情况下，只有无状态的 Bean 才可以在多线程环境下共享，在 Spring 中，绝大部分 Bean 都可以声明为 `singleton` 作用域，因为 Spring 对一些 Bean 中非线程安全状态采用 `ThreadLocal` 进行处理，解决线程安全问题

`ThreadLocal` 和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了 “时间换空间” 的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而 `ThreadLocal` 采用了 “时间换空间” 的方式

`ThreadLocal` 会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。`ThreadLocal` 提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进 `ThreadLocal`

## 3.10 解释 Spring 框架中 bean 的生命周期

在传统的 Java 应用中，bean 的生命周期很简单。使用 Java 关键字 new 进行 `bean` 实例化，然后该 `bean` 就可以使用了。一旦该 `bean` 不再被使用，则由 Java 自动进行垃圾回收。相比之下，Spring 容器中的 `bean` 的生命周期就显得相对复杂多了。正确理解 spring bean 的生命周期非常重要，因为你或许要利用 spring 提供的扩展点来自定义bean 的创建过程。下图展示了 bean 装载到 spring 应用上下文中的一个典型的生命周期过程

![img](md-image/201911012343410.png)

bean 在 spring 容器中从创建到销毁经历了若干阶段，每一阶段都可以针对 Spring 如何管理 bean 进行个性化定制

正如你所见，在 bean 准备就绪之前，bean 工厂执行了若干启动步骤

我们对上图进行详细描述：

- Spring 对 `bean` 进行实例化
- Spring 将值和 `bean` 的引用注入到 `bean` 对应的属性中
- 如果 `bean` 实现了 `BeanNameAware` 接口，Spring 将 `bean` 的 ID 传递给 `setBeanName()` 方法
- 如果 `bean` 实现了 `BeanFactoryAware` 接口，Spring 将调用 `setBeanFactory()` 方法，将 `beanFactory` 容器实例传入
- 如果 `bean` 实现了 `ApplicationContextAware` 接口，Spring 将调用 `setApplicationContext()` 方法，将 `bean` 所在的应用上下文的引用传入进来
- 如果 `bean` 实现了 `BeanPostProcessor` 接口，Spring 将调用它们的 `postProcessBeforeInitialization()` 方法
- 如果 `bean` 实现了 `InitializingBean` 接口，Spring将调用它们的 `after-PropertiesSet()` 方法。类似地，如果bean使用initmethod声明了初始化方法，该方法也会被调用；
- 如果 `bean` 实现了 `BeanPostProcessor` 接口，Spring将调用它们的 `post-ProcessAfterInitialization()` 方法；

此时，`bean` 已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；

如果 `bean` 实现了 `DisposableBean` 接口，Spring将调用它的 `destroy()` 接口方法。同样，如果 `bean` 使用 `destroy-method` 声明了销毁方法，该方法也会被调用。

现在你已经了解了如何创建和加载一个Spring容器。但是一个空的容器并没有太大的价值，在你把东西放进去之前，它里面什么都没有。为了从Spring的DI(依赖注入)中受益，我们必须将应用对象装配进Spring容器中。

## 3.11 哪些是重要的 bean 生命周期方法，能重载吗

有两个重要的 `bean` 生命周期方法，第一个是 `setup`，它是在加载 `bean` 的时候被调用。第二个方法时 `teardown` 它是在容器卸载类的时候被调用

`bean` 标签有两个重要的属性（`init-method` 和 `destory-method`）。用他们你可以自己定制初始化和注销方法。他们也有相应的注解（`@PostConstruct` 和 `@PreDestroy`）

## 3.12 什么是 Spring 的内部 bean？什么是 Spring inner beans

在 Spring 框架中，当一个 `bean` 仅被用作另一个 `bean` 的属性时，它能被声明为一个内部的 `bean`。内部 `bean` 可以用 `setter` 注入”属性“和构造方法注入”构造参数“的方式来实现，内部 `bean` 通常是匿名的，他们的 `scope` 一般是 `prototype`

## 3.13 什么是 bean 装配

装配，或 `bean` 装配是指在 Spring 容器中把 `bean` 组装在一起，前提是容器需要知道 `bean` 的依赖关系，如何通过依赖注入把他们装配到一起

## 3.14 什么是 bean 的自动装配

在 Spring 框架中，在配置文件中设定 bean 的依赖关系是一个很好的机制，Spring 容器能够自动装配相互合作的 `bean`，这意味着容器不需要配置，能通过 `bean` 工厂自动处理 `bean` 之间的协作。这意味着 Spring 可以通过向 `Bean Factory` 中注入的方式自动搞定 `bean` 之间的依赖关系。自动装配可以设置在每个 `bean` 上，也可以设定在特定的 `bean` 上



































