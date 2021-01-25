# 1、web内容回顾

## 1.1 JavaEE三层结构

1. web 层
2. service 层
3. dao 层



## 1.2 MVC思想

1. m：模型
2. v：视图
3. c：控制器



# 2、Hibernate概述

## 2.1 什么是框架

写程序，使用框架之后，帮我们实现一部分功能，使用框架好处，少些一部分代码实现



## 2.2 什么是Hibernate框架

1. Hibernate 框架应用在 JavaEE三层框架中 dao层框架
2. 在 dao 层里面做对数据库 crud 操作，使用 Hibernate 实现crud 操作，Hibernate 底层代码就是 jdbc，Hibernate 对 jdbc 进行封装，使用 Hibernate 好处，不需要写复杂 jdbc代码了，不需要写sql语句实现。
3. Hibernate 开源的轻量级框架



## 2.3 什么是 orm 思想

1. Hibernate使用 orm 思想对数据库进行 crud 操作

2. 在web阶段学习 Javabean ，更正确的叫法 实体类

3. orm：object relational mapping     对象关系映射

   1. 让实体类和数据库表进行——对应关系

      让实体类首先和数据库表对应

      让实体类属性和表里面字段对应

   2. 不需要直接操作数据库表，而操作表对应实体类对象

# 3、Hibernate入门

## 3.1 搭建Hibernate 环境

1. 添加依赖

2. 创建实体类

3. 配置实体类和数据库表（映射关系）

   1. 使用配置文件实现映射关系，创建xml格式的配置文件，映射配置文件名称和位置没有固定要求

      建议：在实体类所在包里面创建，实体类名称 hbm.xml

   2. 配置是 xml 格式，在配置文件中首先引入 xml 约束，在Hibernate里面引入dtd约束

```java
@Data
public class User {
    private int uid;
    private String username;
    private String password;
    private String address;

}
```

配置文件 User.hbm.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <!--配置表和类的映射-->
    <class name="cn.itcast.entity.User" table="t_user">
        <!--配置实体类id和表id对应-->
        <!--Hibernate要求实体类有一个属性唯一值，Hibernate要求表有字段作为唯一值-->
        <id name="uid" column="id">
            <!--设置数据表id增长策略
                native：生成表id值就是主键自动增长
            -->
            <generator class="native"></generator>
        </id>

        <!--配置其他属性和表字段对应-->
        <property name="username" column="username"></property>
        <property name="password" column="password"></property>
        <property name="address" column="address"></property>

    </class>
</hibernate-mapping>
```

4. 创建 hibernate 核心配置文件

   1. 核心配置文件格式 xml，但是核心配置文件名称和数据固定的

      位置：必须在src下面

      名称：必须是 hibernate.cfg.xml

   2. 引入约束

   3. Hibernate 操作过程中，只会加载核心配置文件，其他配置文件不加载

      1. 配置数据库信息
      2. 配置 hibernate 信息
      3. 把配置文件放置在核心配置文件中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--1. 配置数据库信息,必需的-->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver </property>
        <property name="hibernate.connection.url">jdbc:mysql:///hibernate_day1 </property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">1015</property>

        <!--2. 配置 hibernate 信息，可选的-->
        <!--配置底层sql语句-->
        <property name="hibernate.show_sql">true</property>
        <!--输出底层sql语句格式-->
        <property name="hibernate.format_sql">true</property>
        <!--hibernate帮创建表，需要配置之后
            update：如果已经有表，更新，如果没有，创建
        -->
        <property name="hibernate.hbm2ddl.auto">update</property>
        <!--配置数据库方言
            在mysql里面实现分页，关键字limit，只能在myql里面
            在oracle数据库，实现分页rownum
            让hibernate框架识别不同数据库的自己特有的语句
            MySQL5版本之前使用 MySQLDialect，MySQL版本5之后使用过 MySQL5Dialect
        -->
        <property name="hibernate.dialect"> org.hibernate.dialect.MySQL5Dialect </property>

        <!--3. 把配置文件放置在核心配置文件中,必须的-->
        <mapping resource="cn/itcast/entity/User.hbm.xml"/>

    </session-factory>
</hibernate-configuration>
```



## 3.2 添加功能的实现

【测试】

**步骤**：

1. 加载 hibernate 核心配置文件
2. 创建 SessionFactory 对象
3. 使用 SessionFactory 创建 session 对象
4. 开启事务
5. 写具体逻辑 crud 操作
6. 提交事务
7. 关闭资源

```java
package an.itcast.Test;

import cn.itcast.entity.User;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.junit.Test;

/**
 * @Author Cyy
 * @Date 2020/2/26 20:01
 */
public class TestDemo1 {

    @Test
    public void testAdd(){

        //1. 加载 hibernate 核心配置文件
        //在hibernate里面封装对象
        Configuration cfg = new Configuration().configure();

        //2. 创建 SessionFactory 对象
        /**
         * 读取hibernate核心配置文件内容，创建SessionFactory
         * 在此过程中，根据映射关系，在配置数据库里面把表创建
         */
        SessionFactory sessionFactory = cfg.buildSessionFactory();

        //3. 使用 SessionFactory 创建 session 对象
        // 类似于连接
        Session session = sessionFactory.openSession();

        //4. 开启事务
        Transaction tx = session.beginTransaction();

        //5. 写具体逻辑 crud 操作
        //添加功能
        User user = new User("张三","1234","China");
        //调用session的方法实现添加
        session.save(user);

        //6. 提交事务
        tx.commit();

        //7. 关闭资源
        session.close();
        sessionFactory.close();
    }
}
```



# 4、Hibernate 配置文件详解

## 4.1 Hibernate 映射配置文件

1. 映射配置文件名称和位置没有固定要求
2. 映射配置文件中，标签 name 属性值写实体类相关内容
   1. class 标签 name 属性值实体类全路径
   2. id 标签和 property 标签 name 属性值，实体类属性名称
3. id 标签和 property 标签，column 属性可以省略的
   1. 不写column值，表的列值会默认和name属性值一样
4. property 标签type 属性，设置生成字段类型，默认会自动生成对应类型

## 4.2 Hibernate 核心配置文件

1. 配置写位置要求

   ```xml
   <hibernate-configuration>
   	<session-factorty>
       
       </session-factorty>
   </hibernate-configuration>
   ```

2. 配置三部分要求

   1. 数据库部分必须的
   2. hibernate 部分可选的
   3. 映射文件必须的

3. 核心配置文件名和位置固定

   1. 位置：src 下面
   2. 名称：hibernate.cfg.xml



# 5、Hibernate 核心 API

## 5.1 Configuration

```java
Configuration cfg = new Configuration().configure();
```

到 src 下面找到名称 hibernate.cfg.xml 配置文件，创建对象，把配置文件放到对象里面（加载核心配置文件）

## 5.2 SessionFactory

1. 使用 configuration 对象创建 SessionFactory 对象

   1. 创建 SessionFactory 过程中做事情：

      根据核心配置文件中，有数据库配置，有映射文件部分，到数据库里面根据映射关系把表创建

      ```xml
      <property name="hibernate.hbm2ddl.auto">update</property>
      ```

2. 创建 SessionFactory 过程中，这个过程特别耗资源的

   1. 在 hibernate 操作中，建议一个项目一般创建一个 SessionFactory 对象

3. 具体实现

   1. 写工具类，写静态代码块实现

      静态代码块在类加载时候执行，执行一次
      
      ```java
      package cn.itcast.utils;
      
      import org.hibernate.SessionFactory;
      import org.hibernate.cfg.Configuration;
      
      public class HibernateUtils {
      
          private static final Configuration cfg;
          private static final SessionFactory sessionFactory;
      
          //静态代码块
          static {
              //加载核心配置文件
              cfg = new Configuration().configure();
              sessionFactory = cfg.buildSessionFactory();
          }
      
          //提供方法返回SessionFactory
          public static SessionFactory getSessionFactory(){
              return sessionFactory;
          }
      }
      ```

## 5.3 Session

1. Session类似于 jdbc 中 connection
2. 调用 Session 里面不同的方法实现 crud 操作
   1. 添加 save 操作
   2. 修改 update 操作
   3. 删除 delete 操作
   4. 根据 id 查询 get 方法
3. Session 对象单线程对象
   1. Session 对象不能共用，只能自己用

## 5.4 Transaction

1. 事务对象

```java
Transaction tx = session.beginTransaction();
```

2. 事务提交和回滚操作

```java
tx.commit();
tx.rollback();
```

3. 事务概念

   **事务的四个特性**

   原子性、一致性、隔离性、持久性

   - **原子性**： 操作这些指令时，要么全部执行成功，要么全部不执行。 
   - **一致性**： 事务的执行使数据从一个状态转换为另一个状态，但是对于整个数据的完整性保持稳定 
   - **隔离性**： 隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离 
   - **持久性**： 当事务正确完成后，它对于数据的改变是永久性的 