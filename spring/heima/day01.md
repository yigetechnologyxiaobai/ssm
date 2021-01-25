# 1、spring 的概述<font size='3' color='red'>（了解）</font>

- spring是什么
- spring的两大核心
- spring的发展历程和优势
- spring的体系结构

# 2、程序的耦合及解耦

## 2.1 曾经案例中问题

例：连接数据库

```java
/**
*  程序的耦合
*      耦合：程序间的依赖关系
*          包括：
*              类之间的依赖
*              方法间的依赖
*      解耦：
*          降低程序间的依赖关系
*      实际开发中：
*          应该做到：编译期不依赖，运行时才依赖
*      解耦的思想：
*          第一步：使用反射来创建对象，而避免使用new关键字
*          第二步：通过读取配置文件来获取要创建的对象全限定类名		
*/
//1.注册驱动
DriverManager.registerDriver(new com.mysql.jdbc.Driver());
//Class.forName("com.jdbc.mysql.driver");
//2.获取连接
String url = "jdbc:mysql://localhost:3306/eesy";
String username = "root";
String password = "1015";
Connection conn = DriverManager.getConnection(url, username, password);
```



## 2.2 工厂模式解耦

表现层——业务层——持久层

在 bean.properties 文件中写好对应的位置

```properties
#	key-value中，value属性值是对应的实现类的位置
accountService=com.itheima.service.impl.AccountServiceImpl
accountDao=com.itheima.dao.impl.AccountDaoImpl
```

```java
/**
* 一个创建Bean对象的工厂
*
* Bean，在计算机英语中，有可重用组件的含义
* JavaBean：用Java语言编写的可重用组件
*          javabean  >  实体类
*
*    它就是创建我们的service和dao对象的
*
*    第一个：需要一个配置文件来配置我们的service和dao
*              配置的内容，唯一标识=全限定类名(key=value)
*
*    第二个：通过读取配置文件中配置的内容，反射创建对象
*
*    我的配置文件可以是xml也可以说是properties
*/
public class BeanFactory {

   //定义一个Properties对象
   private static Properties props;

   //使用静态代码块为Properties对象赋值
   static {
       try {
           //实例化对象
           props = new Properties();
           //获取properties文件的流对象
           InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
           props.load(in);
           //实例化容器
           beans = new HashMap<String, Object>();
           //取出配置中所有的key
           Enumeration keys = props.keys();//枚举
           //遍历枚举
           while (keys.hasMoreElements()){
               //取出每个key
               String key = keys.nextElement().toString();
               //根据key获得value
               String beanPath = props.getProperty(key);
               //反射创建对象
               Object value = Class.forName(beanPath).newInstance();
               //把key和value存入容器中
               beans.put(key,value);
               
           }
       }catch (Exception ex){
           throw new ExceptionInInitializerError("初始化properties失败");
       }
       
    /**
    * 根据beanName的名称来获取对象（更简便的方法，对象是单例的）
    * @param beanName
    * @return
    */
   public static Object getBean(String beanName){
       return beans.get(beanName);
   }

   /**
  * 根据bean的名称获取bean对象（原先比较繁琐的方法，对象是多例的）
  * @param beanName
  * @return
    */
   /*
   public static Object getBean(String beanName){
       Object bean = null;
       try {
           String beanPath = props.getProperty(beanName);
           bean = Class.forName(beanPath).newInstance();
       }catch (Exception ex){
           ex.printStackTrace();
       }
       return bean;
   }
   */
}
```

```java
/**
* 模拟一个表现层，用于调用业务层
*/
public class Client {
    public static void main(String[] args) {
        //IAccountService as = new AccountServiceImpl();
        IAccountService as = (IAccountService) BeanFactory.getBean("accountService");
        as.SaveAccount();
    }
}
```



# 3、IOC概念和spring中的IOC

## 3.1 IOC（控制反转）

**概念：**

​	控制反转把创建对象的权利交给框架是框架的重要特征，并非面向对象编程的专用术语。它包括依赖注入、和依赖查找

**作用：**

​	削减计算机程序的耦合，解除我们代码中的依赖关系。



## 3.2 spring中基于XML的IOC环境搭建

在 resources 目录下创建 bean.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--把对象的创建交给spring管理-->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>

<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl"></bean>

</beans>
```

```java
/**
* 获取spring的IOC核心容器，并根据id获取对象
*
* @param args
*/
public static void main(String[] args) {

  	//1.获取核心容器对象
  	ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
  	//2.根据id获取Bean对象
  	IAccountService as = (IAccountService) ac.getBean("accountService");
  	IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);

  	System.out.println(as);
  	System.out.println(adao);

}
```



## 3.3 ApplicationContext的三个常用实现类：

- `ClassPathXmlApplicationContext`：它可以加载类路径下的配置文件，要求配置文件必须在类路径下。

- `FileSystemXmlApplicationContext`：它可以加载磁盘任意路径下的配置文件（必须有访问权限）

- `AnnotationConfigApplicationContext`：它是用于读取注解创建容器的



## 3.4 核心容器的两个接口的区别

- ApplicationContext：     单例对象适用

它在构建核心容器时，创建对象采取的政策是采用立即加载的方式，也就是说，只要一读取完配置文件马上就创建配置文件中配置的对象

- BeanFactory：        多例对象适用

它在构建核心容器时，创建对象采取的政策是采用延迟加载的方式，也就是说，什么时候根据id获取对象了，什么时候才真正的创建了

```java
//单例对象
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

//多例对象
BeanFactory factory = new XmlBeanFactory("bean.xml");	//已过时
```



## 3.5 创建bean的三种方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--里面的bean有三种方式-->
<bean>里面内容在下面</bean>

</beans>
```



第一种方式：使用==默认构造函数==创建，在spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。

```java
public class AccountServiceImpl implements IAccountService {

	public AccountServiceImpl(){	//默认构造函数
		
	}

}
```

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
```



第二种方式：使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）

```java
/**
* 虚拟一个工厂类（该类可能是存在于jar包中的，我们无法通过修改源码的方式来提供默认构造函数）
*/
public class InstanceFactory {
	public IAccountService getAccountService(){
  		return new AccountServiceImpl();
	}
}
```

```xml
<bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService">
```



第三种方式：使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器）

```java
/**
* 虚拟一个工厂类（该类可能是存在于jar包中的，我们无法通过修改源码的方式来提供默认构造函数）
*/
public class StaticFactory {
	public static IAccountService getAccountService(){
  		return new AccountServiceImpl();
	}
}
```

```xml
<bean id="accountService" class="com.itheima.factory.StaticFactory" factory-method = "getAccountService"></bean>
```



## 3.6 bean的作用范围

bean标签的scope属性：

作用：

- 用于指定bean的作用范围

取值：

- singleton：单例的（默认值）
- prototype：多例的
- request：作用于web应用的请求范围
- session：作用于web应用的会话范围
- global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，它就是session

```java
public class AccountServiceImpl implements IAccountService {
   	public AccountServiceImpl(){
       		System.out.println("对象创建了");
   }
   	public void SaveAccount() {
       		System.out.println("service中的saveAccount方法执行了");
   	}
   	public void init(){
       		System.out.println("对象初始化了");
   	}
   	public void destroy(){
       		System.out.println("对象销毁了");
   	}
}
```

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"
         scope="prototype" init-method="init" destroy-method="destroy"></bean>
```



## 3.7 bean的生命周期

单例对象    

- 出生：当容器创建时对象出生    
- 活着：只要容器还在，对象一直存活    
- 死亡：容器销毁，对象消亡    
- 总结：单例对象的生命周期和容器相同

多例对象    

- 出生：当我们使用对象时spring框架为我们创建    
- 活着：对象只要是在使用过程中就一直活着    
- 死亡：当对象长时间不用，且没有别的对象引用时，由java的垃圾回收器回收

```java
public static void main(String[] args) {

//1.获取核心容器对象
		ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
/*
	如果想使用多例对象，可以使用
	BeanFactory bf = new XmlBeanFactory("bean.xml");
	目前 XmlBeanFactory 类已过时
*/

/*
	ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
	如果使用ApplicationContext接口进行接收时，因为多态，使用close方法时会报错，所以应该用他本身来接收类
*/
	
  //2.根据id获取Bean对象
	IAccountService as = (IAccountService) ac.getBean("accountService");
	ac.close();

}
```



# 4、依赖注入

## 4.1 Spring中的依赖注入

**依赖注入：**    

- Dependency Injection

**IOC的作用：**   

-  降低程序间的耦合性（依赖关系）

**依赖关系的管理：**    

- 以后都交给spring来维护在当前类需要用到其他类的对象，由spring为我们提供，我们只需要在配置文件中说明

**依赖关系的维护：**    

- 就称之为依赖注入依赖注入：    

**能注入的数据：<font size='3' color='blue'>（有三类）</font>**

- 基本类型和String        
- 其他bean类型（在配置文件中或者注释配置过的bean）        
- 复杂类型/集合类型



**注入的方式，<font size='3' color='blue'>（有三种）</font>**

- 第一种：使用构造函数提供    

```java
//实现类有参数，并且只有一个有参的构造函数
public class AccountServiceImpl implements IAccountService {

	private String name;
	private Integer age;
	private Date birthday;

	public AccountServiceImpl(String name, Integer age, Date birthday) {
   		this.name = name;
   		this.age = age;
   		this.birthday = birthday;
	}
}
```

```xml
	<!--
   构造函数注入：
       使用的标签：constructor-arg
       标签出现的位置：bean标签的内部
       标签中的属性：
           type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
           index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始
           name：用于指定给构造函数中指定，名称的参数赋值                          常用
           —————————以上三个常用语指定给构造函数中那个参数赋值——————————
           value：用于提供基本数据类型和string类型的数据
           ref：用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器中出现过的bean对象
-->

<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
   <constructor-arg name="name" value="张三"></constructor-arg>
   <constructor-arg name="age" value="18"></constructor-arg>
   <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
	 <!-- 调用之后会给对象的参数赋值 -->

<!--配置一个日期对象-->
<bean id="now" class="java.util.Date"></bean>
```



- 第二种：使用set方法提供    

```java
//可以不包含构造方法，但需要有set方法
public class AccountServiceImpl2 implements IAccountService {

	private String name;
	private Integer age;
	private Date birthday;

	public void setName(String name) {
   		this.name = name;
	}

	public void setAge(Integer age) {
  		 this.age = age;
	}

	public void setBirthday(Date birthday) {
   		this.birthday = birthday;
	}
}
```

```xml
	<!--
   set方法注入						常用
   涉及的标签：property
   出现的位置：bean标签的内部
   标签的属性：
       name：用于指定注入时所调用的set方法名称
       value：用于提供基本类型和String类型的数据
       ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象
   优势：
       创建对象时没有明确的限制，可以直接使用默认构造函数
   弊端：
       如果有某个成员必须有值，则获取对象是有可能set方法没有执行。
-->

<bean id="accountService2" class="com.itheima.service.impl.AccountServiceImpl2">
   <property name="name" value="李四"></property>
   <property name="age" value="20"></property>
   <property name="birthday" ref="now"></property>
</bean>
```



- 复杂类型/集合类型

```java
public class AccountServiceImpl3 implements IAccountService {

   private String[] myArr;
   private List<String> myList;
   private Set<String> mySet;
   private Map<String,String> myMap;
   private Properties myProps;

	//参数set方法

}
```

```xml
	<!-- 复杂类型的注入/集合类型的注入
       用于给List结构集合注入的标签：
           list array set
       用于个Map结构集合注入的标签:
           map  props
       结构相同，标签可以互换
   -->
   <bean id="accountService3" class="com.itheima.service.impl.AccountServiceImpl3">
       <property name="myArr">
           <array>
               <value>AAA</value>
               <value>BBB</value>
               <value>CCC</value>
           </array>
       </property>
       <property name="myList">
           <list>
               <value>AAA</value>
               <value>BBB</value>
               <value>CCC</value>
           </list>
       </property>
       <property name="mySet">
           <set>
               <value>AAA</value>
               <value>BBB</value>
               <value>CCC</value>
           </set>
       </property>
       <property name="myMap">
           <map>
               <entry key="test1" value="AAA"></entry>
               <entry key="test2">
                   <value>BBB</value>
               </entry>
           </map>
       </property>
       <property name="myProps">
           <props>
               <prop key="test1">AAA</prop>
               <prop key="test2">BBB</prop>
           </props>
       </property>
   </bean>
```



- 第三种：使用注解提供（day02讲解）