# 1、spring中的JdbcTemplate

  ​		JdbcTemplate的作用

  ​					它就是用于和数据库交互的，实现表的CRUD操作

  ​		如何创建该对象：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
          <property name="dataSource" ref="dataSource"></property>
      </bean>
  
      <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
          <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
          <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
          <property name="username" value="root"></property>
          <property name="password" value="1015"></property>
      </bean>
  </beans>
  ```

  ```java
  //获取容器
  ApplicationContext sc = new ClassPathXmlApplicationContext("bean.xml");
  //获取对象
  JdbcTemplate template = sc.getBean("jdbcTemplate",JdbcTemplate.class);
  ```

  对象中常用的方法：

  ```java
//增删改查
//保存
template.update("insert into account (name,money) values (?,?);","ddd",1000);

//删除
template.update("delete from account where id = ?",6);

//修改
template.update("update account set name=?,money=? where id=?","test",9999,6);

//查询所有

List<Account> accounts = template.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
for(Account account:accounts){
    System.out.println(account);
}

//查询一个
List<Account> accounts = template.query("select * from account where id=?",new BeanPropertyRowMapper<Account>(Account.class),1);
System.out.println(accounts.isEmpty()?"没有结果":accounts.get(0));


//查询返回一行一列（使用聚合函数，但不加group by字句）
Long s = template.queryForObject("select count(*) from account where money > ?",Long.class,1000);
System.out.println(s);
  ```

  

# 2、作业

​		spring基于AOP的事务控制

# 3、spring中的事务控制

## 3.1 基于XML的

```java
//IAccountDao接口的实现类
public class AccountDaoImpl extends JdbcDaoSupport implements IAccountDao {

    @Override
    public Account findAccountById(Integer id) {
       List<Account> accounts = super.getJdbcTemplate().query("select * from account where id=?",new BeanPropertyRowMapper<Account>(Account.class),id);
       return accounts.isEmpty()?null:accounts.get(0);
    }

    @Override
    public Account findAccountByName(String name) {
       List<Account> accounts = super.getJdbcTemplate().query("select * from account where name=?",new BeanPropertyRowMapper<Account>(Account.class),name);
       if (accounts == null){
           return null;
       }
       if(accounts.size()>1){
           throw new RuntimeException("结果不唯一");
       }
       return accounts.get(0);
    }

    public void update(Account account) {
       super.getJdbcTemplate().update("update account set name=?,money=? where id=?",account.getName(),account.getMoney(),account.getId());
    }
}
```

```java
//IAccountService接口的方法
/**
* 账户的业务层实现类
*
* 事务控制应该都是在业务层
*/
public class AccountServiceImpl implements IAccountService{

   private IAccountDao accountDao;

   public void setAccountDao(IAccountDao accountDao) {
       this.accountDao = accountDao;
   }

   @Override
   public Account findAccountById(Integer accountId) {
       return accountDao.findAccountById(accountId);
   }

   @Override
   public void transfer(String sourceName, String targetName, Float money) {
       System.out.println("transfer....");
           //2.1根据名称查询转出账户
           Account source = accountDao.findAccountByName(sourceName);
           //2.2根据名称查询转入账户
           Account target = accountDao.findAccountByName(targetName);
           //2.3转出账户减钱
           source.setMoney(source.getMoney()-money);
           //2.4转入账户加钱
           target.setMoney(target.getMoney()+money);
           //2.5更新转出账户
           accountDao.update(source);

           int i=1/0;

           //2.6更新转入账户
           accountDao.update(target);
   }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xmlns:tx="http://www.springframework.org/schema/tx"
      xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       https://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

   <!--配置账户的持久层-->
   <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
       <property name="dataSource" ref="dataSource"></property>
   </bean>

   <!--配置业务层-->
   <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
       <property name="accountDao" ref="accountDao"></property>
   </bean>

   <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
       <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
       <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
       <property name="username" value="root"></property>
       <property name="password" value="1015"></property>
   </bean>

   <!--
           spring中基于XML的声明式事务控制配置步骤
               1.配置事务管理器
               2.配置事务的通知
                       此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
                       使用tx：advice标签配置事务通知
                           属性：
                               id：给事务或通知起一个唯一标识
                               transaction：给事务通知提供一个事务管理器引用
               3.配置AOP中的通用切入点表达式的对应关系
               4.建立事务通知和切入点表达式的对应关系
               5.配置事务的属性
                       是在事务的通知 tx:advice标签的内部
   -->

   <!--配置事务管理器-->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource"></property>
   </bean>

   <!--配置事务的通知-->
   <tx:advice id="txAdvice" transaction-manager="transactionManager">
       <!-- 配置事务的属性
       isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
       propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
       read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
       timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
       rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
       no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
      -->
       <tx:attributes>
           <tx:method name="*" propagation="REQUIRED" read-only="false"/>
           <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
           <!--特指的比泛指的优先级别高-->
       </tx:attributes>
   </tx:advice>

   <!--配置aop-->
   <aop:config>
       <!--配置切入点表达式-->
       <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
       <!--建立切入点表达式和事务通知的对应关系-->
       <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
   </aop:config>

</beans>
```



## 3.2 基于注解的

```java
//AccountDaoImpl类
package com.itheima.dao.impl;

import com.itheima.dao.IAccountDao;
import com.itheima.domain.Account;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
* 账户的持久层实现类
*/
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {

   @Autowired
   private JdbcTemplate jdbcTemplate;

   @Override
   public Account findAccountById(Integer accountId) {
       List<Account> accounts = jdbcTemplate.query("select * from account where id = ?",new BeanPropertyRowMapper<Account>(Account.class),accountId);
       return accounts.isEmpty()?null:accounts.get(0);
   }

   @Override
   public Account findAccountByName(String accountName) {
       List<Account> accounts = jdbcTemplate.query("select * from account where name = ?",new BeanPropertyRowMapper<Account>(Account.class),accountName);
       if(accounts.isEmpty()){
           return null;
       }
       if(accounts.size()>1){
           throw new RuntimeException("结果集不唯一");
       }
       return accounts.get(0);
   }

   @Override
   public void updateAccount(Account account) {
       jdbcTemplate.update("update account set name=?,money=? where id=?",account.getName(),account.getMoney(),account.getId());
   }
}
```

```java
//AccountServiceImpl类
package com.itheima.service.impl;

import com.itheima.dao.IAccountDao;
import com.itheima.domain.Account;
import com.itheima.service.IAccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

/**
* 账户的业务层实现类
*
* 事务控制应该都是在业务层
*/
@Service("accountService")
@Transactional(propagation= Propagation.SUPPORTS,readOnly=true)//只读型事务的配置
public class AccountServiceImpl implements IAccountService{

   @Autowired
   private IAccountDao accountDao;

   @Override
   public Account findAccountById(Integer accountId) {
       return accountDao.findAccountById(accountId);

   }


   //需要的是读写型事务配置
   @Transactional(propagation= Propagation.REQUIRED,readOnly=false)
   @Override
   public void transfer(String sourceName, String targetName, Float money) {
       System.out.println("transfer....");
       //2.1根据名称查询转出账户
       Account source = accountDao.findAccountByName(sourceName);
       //2.2根据名称查询转入账户
       Account target = accountDao.findAccountByName(targetName);
       //2.3转出账户减钱
       source.setMoney(source.getMoney()-money);
       //2.4转入账户加钱
       target.setMoney(target.getMoney()+money);
       //2.5更新转出账户
       accountDao.updateAccount(source);

       int i=1/0;

       //2.6更新转入账户
       accountDao.updateAccount(target);
   }
}
```



三个config包下的配置类

```java
package config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import javax.sql.DataSource;

/**
* 和连接数据库相关的配置类
*/
public class JdbcConfig {

   @Value("${jdbc.driver}")
   private String driver;

   @Value("${jdbc.url}")
   private String url;

   @Value("${jdbc.username}")
   private String username;

   @Value("${jdbc.password}")
   private String password;

   /**
    * 创建JdbcTemplate
    * @param dataSource
    * @return
    */
   @Bean(name = "jdbcTemplate")
   public JdbcTemplate createJdbcTemplate(DataSource dataSource){
       return new JdbcTemplate(dataSource);
   }

   /**
    * 创建数据源对象
    * @return
    */
   @Bean(name = "dataSource")
   public DataSource createDataSource(){
       DriverManagerDataSource ds = new DriverManagerDataSource();
       ds.setDriverClassName(driver);
       ds.setUrl(url);
       ds.setUsername(username);
       ds.setPassword(password);
       return ds;
   }
}
```

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
* Spring的配置类，相当于bean.xml
*/
@Configuration
@ComponentScan("com.itheima")
@Import({JdbcConfig.class,TransactionConfig.class})
@PropertySource("jdbcConfig.properties")
@EnableTransactionManagement//开启事务的支持——注解
public class SpringConfiguration {

}
```

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

/**
* 和事务相关的
*/
public class TransactionConfig {

   /**
    * 用于创建事务管理对象
    * @param dataSource
    * @return
    */
   @Bean(name = "transactionManager")
   public PlatformTransactionManager createTransactionManager(DataSource dataSource){
       return new DataSourceTransactionManager(dataSource);
   }
}
```



**连接数据库信息**

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy
jdbc.username=root
jdbc.password=1015
```

```java
//测试类
package com.itheima.test;

import com.itheima.service.IAccountService;
import config.SpringConfiguration;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
* 使用Junit单元测试：测试我们的配置
*/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AccountServiceTest {

   	@Autowired
   	private  IAccountService as;

   	@Test
   	public  void testTransfer(){
     	  as.transfer("aaa","bbb",100f);

   	}
}
```

