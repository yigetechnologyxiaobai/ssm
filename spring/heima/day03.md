# 1、完善我们的 account 案例

## 1.1 分析案例中问题

### 1.1.1 转账问题

```java
//在 IAccountDao 接口中定义方法
/**
    * 根据名称查询账户
    * @param accountName
    * @return  如果有唯一的一个结果就返回，如果没有结果就返回null
    *          如果结果集超过一个就抛异常
    */
Account findAccountByName(String accountName);
```

```java
//AccountDaoImpl 实现类中的方法
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
```



**两个工具类：**

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
//AccountServiceImpl实现类
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
```

```java
//测试类
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



# 2、动态代理

## 2.1 对接口的动态代理

```java
/**
* 对生产厂家要求的接口
*/
public interface Iproducer {

   /**
    * 销售
    * @param money
    */
   public void saleProduct(float money);

   /**
    * 售后
    * @param money
    */
   public void afterService(float money);
}
```

```java
public class Producer implements Iproducer {

   /**
    * 销售
    * @param money
    */
   public void saleProduct(float money) {
       System.out.println("销售产品，并拿到钱"+money);
   }

   /**
    * 售后
    * @param money
    */
   public void afterService(float money) {
       System.out.println("提供售后服务"+money);
   }
}
```

```java
public class Client {

   public static void main(String[] args) {
       final Producer producer = new Producer();

       /**
        * 动态代理 ：
        *      特点：字节码随用随创建，随用随加载
        *      作用：不修改源码的基础上对方法增强
        *      分类：
        *              基于接口的动态代理
        *              基于子类的动态代理
        *      基于接口的动态代理：
        *               涉及的类：Proxy
        *               提供者：JDK官方
        *      如何创建代理对象：
        *               使用Proxy类中的newProxyInstance方法
        *      创建代理对象的要求：
        *               被代理类最少实现一个接口，如果没有则不能使用
        *      newProxyInstance方法的参数：
        *                ClassLoader：类加载器
        *                    它是用于加载代理对象字节码的。和被代理对象使用相同的类加载器。固定写法。
        *                Class[]：字节码数组
        *                    它是用于让代理对象和被代理对象有相同方法。固定写法。
        *                InvocationHandler：用于提供增强的代码
        *                    它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。
        *                    此接口的实现类都是谁用谁写。
        */
       Iproducer proxyProducer = (Iproducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
               producer.getClass().getInterfaces(),
               new InvocationHandler() {
                   /**
                    * 作用：执行被代理对象的任何接口方法都会经过该方法
                    * @param proxy     代理对象的引用
                    * @param method    当前执行的方法
                    * @param args      当前执行方法所需的参数
                    * @return          和被代理对象方法有相同的返回值
                    * @throws Throwable
                    */
                   @Override	//方法里提供增强的代码
                   public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                       Object returnValue = null;

                       //1.获取方法执行的参数
                       Float money = (Float) args[0];
                       //2.判断当前方法是不是销售
                       if ("saleProduct".equals(method.getName())){
                           returnValue = method.invoke(producer,money*0.8f);
                       }
                       return returnValue;
                   }
               });

       proxyProducer.saleProduct(10000f);
   }
}
```



## 2.2 动态代理另一种实现方式

**对类的动态代理**

```java
public class Producer {

/**
    * 销售
    * @param money
    */
   public void saleProduct(float money) {
       System.out.println("销售产品，并拿到钱"+money);
   }

   /**
    * 售后
    * @param money
    */
   public void afterService(float money) {
       System.out.println("提供售后服务"+money);
   }
}
```

```java
public class Client {

   public static void main(String[] args) {
       final Producer producer = new Producer();

       /**
        * 动态代理 ：
        *      特点：字节码随用随创建，随用随加载
        *      作用：不修改源码的基础上对方法增强
        *      分类：
        *              基于接口的动态代理
        *              基于子类的动态代理
        *      基于子类的动态代理：
        *               涉及的类：Enhancer
        *               提供者：第三方cglib库
        *      如何创建代理对象：
        *               使用Enhance类中的create方法
        *      创建代理对象的要求：
        *               被代理类最少实现一个接口，如果没有则不能使用
        *      create方法的参数：
        *                Class：字节码
        *                    它是用于被代理对象的字节码
        *                Callback：用于提供增强的代码
        *                    它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。
        *                    此接口的实现类都是谁用谁写。
        *                    我们一般写的都是该接口的子接口实现类：MethodInterceptor
        */
       Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
           /**
            *
            * @param o
            * @param method
            * @param objects
            *      以上三个参数和基于接口的动态代理中invoke方法的参数是一样的
            * @param methodProxy   ： 当前执行方法的代理对象
            * @return
            * @throws Throwable
            */
           @Override
           public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

               Object returnValue = null;

               Float money = (Float)objects[0];
               if("saleProduct".equals(method.getName())){
                   returnValue = method.invoke(producer,money*0.8f);
               }
               return returnValue;

           }
       });

       cglibProducer.saleProduct(1000f);
   }
}
```



# 3、解决案例中的问题

```java
//IAccount接口的实现类
public class AccountServiceImpl implements IAccountService {

    private IAccountDao accountDao;

    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public List<Account> findAll() {
        List<Account> accounts = accountDao.findAll();
        return accounts;
    }

    public Account findAccountById(Integer id) {
        Account account = accountDao.findAccountById(id);
        return account;
    }

    @Override
    public void saveAccount(Account account) {
        accountDao.saveAccount(account);
    }

    @Override
    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }

    @Override
    public void deleteAccount(Integer id) {
        accountDao.deleteAccount(id);
    }

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
public class BeanFactory {

   private IAccountService accountService;

   private TransactionManager txManager;

   public void setTxManager(TransactionManager txManager) {
       this.txManager = txManager;
   }

   public final void setAccountService(IAccountService accountService) {
       this.accountService = accountService;
   }

   /**
    * 获取Service代理对象
    * @return
    */
   public IAccountService getAccountService() {
       return (IAccountService) Proxy.newProxyInstance(accountService.getClass().getClassLoader(),
               accountService.getClass().getInterfaces(),
               new InvocationHandler() {
                   /**
                    * 添加事务的支持
                    * @param proxy
                    * @param method
                    * @param args
                    * @return
                    * @throws Throwable
                    */
                   @Override
                   public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                       Object returnValue = null;
                       try{
                           //1.开启事务
                           txManager.beginTransaction();
                           //2.执行操作
                           returnValue = method.invoke(accountService,args);
                           //3.提交事务
                           txManager.commit();
                           //4.返回结果
                           return returnValue;
                       }catch (Exception e){
                           //5.回滚操作
                           txManager.rollback();
                           throw new RuntimeException(e);
                       }finally {
                           //6.释放连接
                           txManager.release();
                       }
                   }
               });
   }

}
```

```java
/**
* 使用junit单元测试，测试我们的配置
*/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class AccountServiceTest {

   @Autowired
   @Qualifier("proxyAccountService")
   private IAccountService as ;

   @Test
   public void testAccount() {
       as.transfer("bbb","aaa",100F);
   }
}
```



# 4、AOP的概念

# 5、spring 中的AOP相关术语

# 6、spring 中基于XML和注解的AOP配置

## 6.1 基于XML的配置

```java
/**
* 账户的业务层接口
*/
public interface IAccountService {

   /**
    * 模拟保存操作
    */
   void saveAccount();

   /**
    * 模拟更新操作
    * @param i
    */
   void updateAccount(int i);

   /**
    * 模拟删除操作
    * @return
    */
   int deleteAccount();

}
```

```java
public class AccountServiceImpl implements IAccountService {
   @Override
   public void saveAccount() {
		//    int i = 1/0;
       System.out.println("执行了保存");
   }

   @Override
   public void updateAccount(int i) {
       System.out.println("执行了修改"+i);
   }

   @Override
   public int deleteAccount() {
       System.out.println("执行了删除");
       return 0;
   }
}
```

```java
/**
* 用于记录日志的工具类，它里面是提供了公共的代码
*/
public class Logger {

   /**
    * 前置通知
    */
   public void beforePrintLog(){
       System.out.println("前置通知：beforePrintLog方法...");
   }

   /**
    * 后置通知
    */
   public void afterRuningPrintLog(){
       System.out.println("后置通知：afterRuningPrintLog方法...");
   }

   /**
    * 异常通知
    */
   public void afterThrowingPrintLog(){
       System.out.println("异常通知：afterThrowingPrintLog方法...");
   }

   /**
    * 最终通知
    */
   public void afterPrintLog(){
       System.out.println("最终通知：afterPrintLog方法...");
   }

   /**
    * 环绕通知
    * 问题：
    *      当我们配置了环绕通知之后，切入点方法没有执行，而通知方法执行了
    * 分析：
    *      通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码中没有。
    * 解决：
    *      Spring框架为我们提供了一个接口：ProceedingJoinPoint。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。
    *      该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。
    *
    * spring中的环境通知：
    *      它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。
    */
   public Object aroundPringLog(ProceedingJoinPoint pjp){
       Object obj = null;
       try {
           Object[] args = pjp.getArgs();

           System.out.println("环绕通知：afterPrintLog方法...前置");

           obj = pjp.proceed(args);

           System.out.println("环绕通知：afterPrintLog方法...后置");

           return obj;
       }catch (Throwable e){
           System.out.println("环绕通知：afterPrintLog方法...异常");
           throw new RuntimeException(e);
       }finally {
           System.out.println("环绕通知：afterPrintLog方法...最终");
       }
   }
}
```

```xml
<!--bean.xml文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

   <!--配置spring的IOC，把service对象配置尽量-->
   <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>

   <!--配置Logger类-->
   <bean id="logger" class="com.itheima.utils.Logger"></bean>

   <!--配置AOP-->
   <aop:config>

       <!--
           配置切入点表达式，id属性用于指定表达式的唯一标识，expression属性用于指定表达式内容
           此标签写在aop:aspect标签内部职能当前切面使用。
           它还可以写在aop:aspect外面，此时就变成了所有切面可用
       -->
       <aop:pointcut id="pt1" expression="execution(void com.itheima.service.impl.*.*(..))"></aop:pointcut>

       <aop:aspect id="LogAdvice" ref="logger">
           <!--配置前置通知,在切入点方法执行之前执行
           <aop:before method="beforePrintLog" pointcut-ref="pt1"></aop:before>-->

           <!--配置后置通知，在切入点方法正常执行之后执行，它和异常通知永远只能执行一个
           <aop:after-returning method="afterRuningPrintLog" pointcut-ref="pt1"></aop:after-returning>-->

           <!--配置异常通知，在切入点方法执行产生异常之后执行，它和后置通知永远只能执行一个
           <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>-->

           <!--配置最终通知-，无论切入点方法是否正常执行它都会在其后面执行
           <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>-->

           <!--环绕通知，具体写在Logger类中-->
           <aop:around method="aroundPringLog" pointcut-ref="pt1"></aop:around>
       </aop:aspect>
   </aop:config>
</beans>
```

```java
public class TestAOP {

   @Test
   public void testLogger() {
       ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
       IAccountService as = (IAccountService) ac.getBean("accountService");
       as.saveAccount();
   }
}
```

## 6.2 基于注解的配置

```java
/**
* 账户的业务层接口
*/
public interface IAccountService {

   /**
    * 模拟保存操作
    */
   void saveAccount();

   /**
    * 模拟更新操作
    * @param i
    */
   void updateAccount(int i);

   /**
    * 模拟删除操作
    * @return
    */
   int deleteAccount();
}
```

```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
   @Override
   public void saveAccount() {
//        int i = 1/0;
       System.out.println("执行了保存");
   }

   @Override
   public void updateAccount(int i) {
       System.out.println("执行了修改"+i);
   }

   @Override
   public int deleteAccount() {
       System.out.println("执行了删除");
       return 0;
   }
}
```

```java
/**
* 用于记录日志的工具类，它里面是提供了公共的代码
*/
@Component("logger")
@Aspect//表示当前类是一个切面类
public class Logger {

   @Pointcut("execution(* com.itheima.service.impl.*.*(..))")
   private void pt1(){}

   /**
    * 前置通知
    */
//    @Before("pt1()")
   public void beforePrintLog(){
       System.out.println("前置通知：beforePrintLog方法...");
   }

   /**
    * 后置通知
    */
//    @AfterReturning("pt1()")
   public void afterRuningPrintLog(){
       System.out.println("后置通知：afterRuningPrintLog方法...");
   }

   /**
    * 异常通知
    */
//    @AfterThrowing("pt1()")
   public void afterThrowingPrintLog(){
       System.out.println("异常通知：afterThrowingPrintLog方法...");
   }

   /**
    * 最终通知
    */
//    @After("pt1()")
   public void afterPrintLog(){
       System.out.println("最终通知：afterPrintLog方法...");
   }

   @Around("pt1()")
   public Object aroundPringLog(ProceedingJoinPoint pjp){
       Object obj = null;
       try {
           Object[] args = pjp.getArgs();

           System.out.println("环绕通知：afterPrintLog方法...前置");

           obj = pjp.proceed(args);

           System.out.println("环绕通知：afterPrintLog方法...后置");

           return obj;
       }catch (Throwable e){
           System.out.println("环绕通知：afterPrintLog方法...异常");
           throw new RuntimeException(e);
       }finally {
           System.out.println("环绕通知：afterPrintLog方法...最终");
       }
   }
}
```

```xml
<!--bean.xml文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

   <!--配置spring创建容器时要扫描的包-->
   <context:component-scan base-package="com.itheima"></context:component-scan>

   <!--配置spring开启注解AOP的支持-->
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

```java
public class TestAOP {

   @Test
   public void testLogger() {
       ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
       IAccountService as = (IAccountService) ac.getBean("accountService");
       as.saveAccount();
   }
}
```