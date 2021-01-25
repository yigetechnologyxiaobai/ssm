# 1、spring中IOC的常用注解

## 1.1 使用注解实现关联效果

```xml
<!--包含context的依赖-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd">

<!--告知spring在创建容器时要扫描的包，配置所需要的标签不是在beans的约束中，而是一个名称为context名称空间和约束中-->
<context:component-scan base-package="com.itheima"></context:component-scan>

</beans>
```

```java
@Component("accountService")//注解实现效果
public class AccountServiceImpl implements IAccountService {

	public void SaveAccount() {
       accountDao.saveAccount();
	}
}
```

```java
public static void main(String[] args) {

   ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
   IAccountService as = ac.getBean("accountService",IAccountService.class);
   System.out.println(as);

}
```

## 1.2 常用的几个注解

### 1.2.1 用于创建对象的

他们的作用就和在XML配置文件中编写一个<bean>标签实现的功能是一样的

​		<font color='gree'>注：使用效果在上面注解实现关联</font>

- ##### Component：

  - 作用：

    ​	用于把当前类对象存入spring容器中

  - 属性：

    ​	value：用于指定bean的id，当我们不写时，它的默认值是当前类名，且首字母小写

- ##### Controller：<font size='3' color='qual'>(一般用在表现层)</font>

- ##### Service：<font size='3' color='qual'>(一般用在业务层)</font>

- ##### Repository：<font size='3' color='qual'>(一般用在持久层)</font>

​        以上三个注解他们的作用和属性与Component是一模一样。

​        他们三个是spring框架为我们提供明确的三层使用的注解，使我们的三层对象更加清晰

### 1.2.2 用于注入数据的

他们的作用就和在XML配置文件中的bean标签中写一个\<property>标签的作用是一样的

- ##### Autowired：

  - ###### 作用：

    - 自动按照类型注入。只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功
    - 如果ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错。
    - 如果Ioc容器中有多个类型匹配时：

  - ###### 出现位置：

    ​		可以是变量上，也可以是方法上

  - ###### 细节：

    ​		在使用注解注入时，set方法就不是必须的了。

  ```java
  @Component("accountService")
  public class AccountServiceImpl implements IAccountService {
  
      @Autowired
      private IAccountDao accountDao = null;	//初始默认值为null，加注释以后才能使用
  
      public void SaveAccount() {
          accountDao.saveAccount();
      }
  }
  ```

  ```java
  @Repository("accountDao")
  public class AccountDaoImpl implements IAccountDao {
  
      public void saveAccount() {
          System.out.println("保存了用户");
      }
  }
  ```

  

- **Qualifier：**

  - 作用：

    在按照类中注入的基础之上再按照名称注入。它在给类成员注入时不能单独使用。但是在给方法参数注入时可以

  - 属性：

    value：用于指定注入bean的id


- **Resource：**

  - 作用：

    直接按照bean的id注入。他可以独立使用

  - 属性：

    name：用于指定bean的id

​		<font color='orange'>以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。*      另外，集合类型的注入只能通过XML来实现。</font>

- **Value**
  - ###### 作用：

    用于注入基本类型和String类型的数据

  - ###### 属性：

    value：用于指定数据的值。它可以使用spring中SpEL(也就是spring的el表达式）

    SpEL的写法：${表达式}



### 1.2.3 用于改变作用范围的

​		他们的作用就和在bean标签中使用scope属性实现的功能是一样的

- ##### Scope

  - ###### 作用：

    ​		用于指定bean的作用范围

  - ###### 属性：

    ​		value：指定范围的取值。常用取值：

    - singleton 	（单例）

- prototype    （多例）



### 1.2.4 和生命周期相关	<font color='yellow' size='3'>（了解）</font>

他们的作用就和在bean标签中使用init-method和destroy-method的作用是一样的
​	

**PreDestroy**

- 作用：

  ​		用于指定销毁方法		

**PostConstruct**

- 作用：

  ​		用于指定初始化方法

  

# 2、使用xml方式和注解方式实现单表的CRUD

<font color='qual'>持久层技术选择：	dbutils</font>

```java
//账户类
public class Account implements Serializable {

   private int id;
   private String name;
   private float money;

  get、set、toString的方法
}    
```

```java
//业务层接口
public interface IAccountService {

   List<Account> findAll();

   Account findAccountById(Integer id);

   void saveAccount(Account account);

   void updateAccount(Account account);

   void deleteAccount(Integer id);

}
```

```java
//业务层实现类
public class AccountServiceImpl implements IAccountService {

   private IAccountDao accountDao;

   public void setAccountDao(IAccountDao accountDao) { this.accountDao = accountDao; }

   public List<Account> findAll() { return accountDao.findAll(); }

   public Account findAccountById(Integer id) { return accountDao.findAccountById(id); }

   public void saveAccount(Account account) { accountDao.saveAccount(account); }

   public void updateAccount(Account account) { accountDao.updateAccount(account); }

   public void deleteAccount(Integer id) { accountDao.deleteAccount(id); }
}
```

```java
//持久层
public interface IAccountDao {

   List<Account> findAll();

   Account findAccountById(Integer id);

   void saveAccount(Account account);

   void updateAccount(Account account);

   void deleteAccount(Integer id);
}
```

```java
public class AccountDaoImpl implements IAccountDao {

   private QueryRunner runner;

   public void setRunner(QueryRunner runner) { this.runner = runner; }

  //IAccountDao中的抽象方法的实现
}
```

```xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置Service-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
       <!--注入dao-->
       <property name="accountDao" ref="accountDao"></property>
    </bean>

    <!--配置Dao对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
       <!--注入QueryRunner-->
       <property name="runner" ref="runner"></property>
    </bean>

    <!--配置QueryRunner-->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
       <!--注入数据源-->
       <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
       <!--连接数据库的必备信息-->
       <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
       <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/eesy"></property>
       <property name="user" value="root"></property>
       <property name="password" value="1015"></property>
    </bean>

</beans>
```

# 3、纯注解的方式实现 IOC 案例

spring的一些新注解使用

```java
//只是在类上和属性上增加了一些注解
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {

	@Autowired
	private QueryRunner runner;

//增删改查方法

}
```

```java
//同上
@Service("accountService")
public class AccountServiceImpl implements IAccountService {

	@Autowired
	private IAccountDao accountDao;

//增删改查方法

}
```



**spring中的新注释**      

- Configuration          

  - 作用：指定当前类是一个配置类*      

- ComponentScan          

  - 作用：

    ​			用于通过注解指定spring在创建容器时要扫描的包*          

  - 属性：              

    ​			value：它和basePackages的作用是一样的，都是用于指定创建容器时要扫描的包                      

    我们使用此注解就等同于在xml中配置了：

    ```
    <context:component-scan base-package="com.itheima"></context:component-scan>
    ```

- Bean          

  - 作用：

    ​			用于把当前方法的返回值作为bean对象存入spring的ioc容器中          

  - 属性：             

     			name：用于指定bean的id，当不写时，默认值是当前方法的名称

```java
//	该类是一个配置类，它的作用和bean.xml是一样的
@Configuration
@ComponentScan(basePackages = "com.itheima")
public class SpringConfiguration {

   /**
       * 用于创建一个QueryRunner对象
       * @param dataSource
       * @return
   */
   @Bean(name = "runner")
   @Scope("prototype")
   public QueryRunner createQueryRunner(DataSource dataSource){
          return new QueryRunner(dataSource);
   }

   @Bean(name = "dataSource")
   public DataSource createDataSource(){
          try {
              ComboPooledDataSource ds = new ComboPooledDataSource();
              ds.setDriverClass("com.mysql.jdbc.Driver");
              ds.setJdbcUrl("jdbc:mysql://localhost:3306/eesy");
              ds.setUser("root");
              ds.setPassword("1015");
              return ds;
          }catch (Exception e){
              throw new RuntimeException(e);
          }
   }

}
```



测试方法

```java
public class AccountServiceImpl {

   private ApplicationContext ac = null;
   private IAccountService as = null;

   @Before
   public void init(){
          //使用纯注释时要使用ApplicationContext的另一个实现类    AnnotationConfigApplicationContext
          ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
          as = ac.getBean("accountService", IAccountService.class);
   }

   @Test
   public void testFindAll() {							//查询所有用户

          List<Account> accounts = as.findAll();
          for(Account account:accounts){
              System.out.println(account);
          }
   }

   @Test
   public void testSaveAccount() {						//保存用户
          Account account = new Account();
          account.setMoney(5000);
          account.setName("ddd");
          as.saveAccount(account);
   }
}
```



## 3.1 不含xml的纯注解写法

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy
jdbc.username=root
jdbc.password=1015
```

```java
/**
* 和spring连接数据库相关的配置类
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
        * 用于创建一个QueryRunner对象
        * @param dataSource
        * @return
        */
       @Bean(name = "runner")
       @Scope("prototype")
       public QueryRunner createQueryRunner(@Qualifier("ds1") DataSource dataSource){
           return new QueryRunner(dataSource);
       }

       @Bean(name = "ds1")
       public DataSource createDataSource1(){
           try {
               ComboPooledDataSource ds = new ComboPooledDataSource();
               ds.setDriverClass(driver);
               ds.setJdbcUrl(url);
               ds.setUser(username);
               ds.setPassword(password);
               return ds;
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }

       @Bean(name = "ds2")
       public DataSource createDataSource2(){
           try {
               ComboPooledDataSource ds = new ComboPooledDataSource();
               ds.setDriverClass(driver);
               ds.setJdbcUrl(url);
               ds.setUser(username);
               ds.setPassword(password);
               return ds;
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }
}
```

```java
@Configuration
@ComponentScan(basePackages = "com.itheima")
@Import(JdbcConfig.class)
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {

}
```



该类是一个配置类，它的作用和bean.xml是一样的

​	spring中的新注释      

- Configuration          

  - 作用：指定当前类是一个配置类          
  - 细节：当配置类作为AnnotationConfigApplicationContext对象创建的参数时，可以不写     

- ComponentScan         

  - 作用：用于通过注解指定spring在创建容器时要扫描的包          

  - 属性：

    - value：它和basePackages的作用是一样的，都是用于指定创建容器时要扫描的包 

      ​	我们使用此注解就等同于在xml中配置了                          

      ```xml
      <context:component-scan base-package="com.itheima"></context:component-scan>
      ```

- Bean

  - 作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中          

  - 属性：              

    - name：用于指定bean的id，当不写时，默认值是当前方法的名称          

    - 细节：当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。 

      ​				查找的方式和Autowired注解的作用是一样的***      **

- Import

  - 作用：用于导入其他的配置类           
  - 属性：               
    - value：用于指定其他的配置类的字节码                          
    - 当我们使用Import的注解之后，有Import注解的类就是父配置类，而导入的就是子配置类

- PropertySource

  - 作用：用于指定properties文件的位置            
  - 属性：                  
    - value：指定文件的名称和路径。
      - 关键字：classpath：表示类路径下

```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {

       @Autowired
       private IAccountDao accountDao;

       public List<Account> findAll() {
           return accountDao.findAll();
       }

       public Account findAccountById(Integer id) {
           return accountDao.findAccountById(id);
       }

       public void saveAccount(Account account) {
           accountDao.saveAccount(account);
       }

       public void updateAccount(Account account) {
           accountDao.updateAccount(account);
       }

       public void deleteAccount(Integer id) {
           accountDao.deleteAccount(id);
       }
}
```

```java
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {

       @Autowired
       private QueryRunner runner;

       public List<Account> findAll() {
           try{
               return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }

       public Account findAccountById(Integer id) {
           try{
               return runner.query("select * from account where id=?",new BeanHandler<Account>(Account.class),id);
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }

       public void saveAccount(Account account) {
           try{
               runner.update("insert into account(name,money) values(?,?)",account.getName(),account.getMoney());
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }

       public void updateAccount(Account account) {
           try{
               runner.update("update account set name=?,money=? where id=?",account.getName(),account.getMoney(),account.getId());
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }

       public void deleteAccount(Integer id) {
           try{
               runner.update("delete from account where id=?",id);
           }catch (Exception e){
               throw new RuntimeException(e);
           }
       }
}
```

```java
public class AccountServiceTest {

       private ApplicationContext ac = null;
       private IAccountService as = null;

       @Before
       public void init(){
           ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
           as = ac.getBean("accountService", IAccountService.class);
       }

       @Test
       public void testFindAll() {

           List<Account> accounts = as.findAll();
           for(Account account:accounts){
               System.out.println(account);
           }
       }
}
```



  

# 4、spring和junit整合

1. 应用程序的入口

   ​		main方法

2. junit单元测试中，没有main方法也能执行

   - junit 集成了一个main方法
   - 该方法就会判断当前测试类中哪些方法有 @Test 注解
   - junit就会让有Test注解的方法执行

3. junit不会管我们是否采用spring框架

   - 在执行测试方法时，junit根本不知道我们是不是使用了spring框架
   - 所以也就不会为我们读取配置文件/配置类创建spring核心容器

4. 由以上三点可知：

   - 当测试方法执行时，没有IOC容器，就算写了Autowired注解，也无法实现注入



- 使用junit单元测试，测试我们的配置

- Spring整合junit的配置      

  - 1.导入spring整合junit的jar（坐标）      

  - 2.使用junit提供的一个注解把原有的main方法替换了，替换成spring提供的          

    - @RunWith

  - 3.告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置          

    - @ContextConfiguration           

      - locations：指定xml文件的位置，加上classpath关键字，表示在类路径下      
      - classes：指定注解类所在的位置

      当我们使用spring 5.x版本的使用，要求junit的版本必须是4.12及以上

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AccountServiceTest {

	@Autowired
	private IAccountService as = null;

	@Test
	public void testFindAll() {

  		List<Account> accounts = as.findAll();
  		for(Account account:accounts){
                System.out.println(account);
 		 }
	}
}
```



5. 转账事务操作

提供两个工具类

```java
/**
* 和事务管理相关的工具类，它包含了：开启事务、提交事务、回滚事务和释放连接
*/
public class TransactionManager {

   private ConnectionUtils connectionUtils;

   public void setConnectionUtils(ConnectionUtils connectionUtils) {
       this.connectionUtils = connectionUtils;
   }

   /**
    * 开启事务
    */
   public void beginTransaction(){
       try {
           connectionUtils.getThreadConnection().setAutoCommit(false);
       }catch (Exception e){
           e.printStackTrace();
       }
   }

   /**
    * 提交事务
    */
   public void commit(){
       try {
           connectionUtils.getThreadConnection().commit();
       }catch (Exception e){
           e.printStackTrace();
       }
   }

   /**
    * 回滚事务
    */
   public void rollback(){
       try {
           connectionUtils.getThreadConnection().rollback();
       }catch (Exception e){
           e.printStackTrace();
       }
   }

   /**
    * 释放连接
    */
   public void release(){
       try {
           connectionUtils.getThreadConnection().close();//还回连接池
           connectionUtils.removeConnection();
       }catch (Exception e){
           e.printStackTrace();
       }
   }

}
```

```java
public class ConnectionUtils {

   private ThreadLocal<Connection> t1 = new ThreadLocal<Connection>();

   private DataSource dataSource;

   public void setDataSource(DataSource dataSource) {
       this.dataSource = dataSource;
   }

   public Connection getThreadConnection(){
       //1.先从ThreadLocal上获取
       Connection conn = t1.get();
       try {
           //2.判断当前线程上是否有连接
           if (conn == null){
               //3.从数据源上获取一个连接，并且存入ThreadLocal中
               conn = dataSource.getConnection();
               t1.set(conn);
           }
           return conn;
       }catch (Exception e){
           throw new RuntimeException(e);
       }
   }

   /**
    * 把连接和线程解绑
    */
   public void removeConnection(){
       t1.remove();
   }

}
```

```java
public class AccountDaoImpl implements IAccountDao {

   private QueryRunner runner;
   private ConnectionUtils connectionUtils;

   public void setRunner(QueryRunner runner) {
       this.runner = runner;
   }

   public void setConnectionUtils(ConnectionUtils connectionUtils) {
       this.connectionUtils = connectionUtils;
   }
	 @Override
   public Account findAccountByName(String accountName) {
       try{
           List<Account> accounts = runner.query(connectionUtils.getThreadConnection(),"select * from account where name = ?", new BeanListHandler<Account>(Account.class), accountName);
           if(accounts == null || accounts.size() == 0){
               return null;
           }
           if(accounts.size() > 1){
               throw new RuntimeException("数据错误，查询结果不唯一");
           }
           return accounts.get(0);
       }catch (Exception e) {
           throw new RuntimeException(e);
       }
   }
}
```

```java
public class AccountServiceImpl implements IAccountService {

   private IAccountDao accountDao;
   private TransactionManager txManager;

   public void setTxManager(TransactionManager txManager) {
       this.txManager = txManager;
   }

   public void setAccountDao(IAccountDao accountDao) {
       this.accountDao = accountDao;
   }
   public void transfer(String sourceName, String targetMoney, Float money) {

       try{
           //1.开启事务
           txManager.beginTransaction();
           //2.执行操作

           //2.1.根据名称查询转出账户
           Account source = accountDao.findAccountByName(sourceName);
           //2.2.根据名称查询转入账户
           Account target = accountDao.findAccountByName(targetMoney);
           //2.3.转出账户减钱
           source.setMoney(source.getMoney()-money);
           //2.4.转入账户加钱
           target.setMoney(target.getMoney()+money);
           //2.5.更新转出账户
           accountDao.updateAccount(source);
           //2.6.更新转入账户
           accountDao.updateAccount(target);

           int i=1/0;

           //3.提交事务
           txManager.commit();
           //4.返回结果
       }catch (Exception e){
           //5.回滚操作
           txManager.rollback();
           e.printStackTrace();
       }finally {
           //6.释放连接
           txManager.release();
       }
   }

}
```

```java
/**
* 使用junit单元测试，测试我们的配置
*/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class AccountServiceImpl {

   @Autowired
   private IAccountService as ;

   @Test
   public void testAccount() {
       as.transfer("aaa","bbb",100F);
   }
}
```

