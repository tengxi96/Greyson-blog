# Spring学习笔记

## 一、Spring入门案例

1. 通过 Idea 创建 maven 项目
2. 配置 Spring 配置文件 ApplicationContext.xml
3. 编写接口及实现类
- IaccountDao

        /**
         * 账户的持久层接口
         */
        public interface IAccountDao {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }

- IaccountService

        /**
         * 账户业务层的接口
         */
        public interface IAccountService {
        
            /**
             * 模拟保存账户
             */
            void saveAccount();
        }

- AccountDaoImpl

        /**
         * 账户的持久层实现类
         */
        public class AccountDaoImpl implements IAccountDao {
        
            public  void saveAccount(){
        
                System.out.println("保存了账户");
            }
        }

- AccountServiceImpl

        /**
         * 账户的业务层实现类
         */
        public class AccountServiceImpl implements IAccountService {
        
            private IAccountDao accountDao = new AccountDaoImpl();
        
            public void  saveAccount(){
                accountDao.saveAccount();
            }
        }

4. 编写测试类 Client

    ```
    /**
     * 模拟一个表现层，用于调用业务层
     */
    public class Client {
        /**
         *
         *获取IOC的核心容器，并根据id获取对象
         * @param args
         */
        public static void main(String[] args) {
            ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");
            // 两种不同的方式获取Bean对象
            IAccountService as = (IAccountService) ac.getBean("accountService");
            IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
            System.out.println(as);
            System.out.println(adao);
    //        as.saveAccount();
        }
    }
    ```

    

ApplicationContext的三个常用实现类：

- ClassPathXmlApplicationContext： 它可以加载路径下的配置文件，要求配置文件必须在路径下，否则加载不了

        ApplicationContext ac = new ClassPathXmlApplicationContext("bea
        ns.xml");

- FileSyetemXmlApplicationContext：它可以加载磁盘下任意路径下的配置文件（必须有访问权限）

    加载方式如下：

        ApplicationContext ac = new FileSystemXmlApplicationContext("C:\\user\\greyson\\...")

- AnnotationConfigApplicationContext：它是用于读取注解创建容器的

核心容器的两个接口引发出来的问题

- ApplicationContext：它在创建核心容器时，创建对象采取的策略是采用立即加载的方式，也就是说，只要一读取完配置文件就马上创建配置文件中配置的对象
    - 单例对象适用
    - 开发中常采用此接口
- BeanFactory: 它在构建核心容器时，创建对象的策略是采用延迟加载的方式，什么时候获取 id 对象了，什么时候就创建对象。
    - 多例对象适用

## 二、Spring 中 Bean 的细节

### 1.三种创建 bean 对象的方式

(1) 使用默认构造函数创建

在spring的配置文件中，使用 id 和 class 属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建 bean 对象，此时如果类中没有默认构造函数，则对象无法创建。

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl"></bean>

(2) 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入 Spring容器）,如下

    /**
     *模拟一个工厂类，该类可能存在于jar包中，无法通过修改源码的方式来提供默认构造函数
     * 
     */
    public class InstanceFactory {
        public IAccountService getAccountService() {
            return new AccountServiceImpl();
        }
    }

配置方式如下：

    <bean id = "instanceFactory" class = "com.itheima.factory.InstanceFactory"></bean>
        <bean id = "accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>

(3) 使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器），如下：

    public class StaticFactory {
        public  static IAccountService getAccountService() {
    
            return new AccountServiceImpl();
        }
    }

配置方式如下：

    <bean id = "accountService" class = "com.itheima.factory.StaticFactory" factory-method="getAccountService"></bean>

### 2.bean 的作用范围调整

(1) bean 标签的 scope 属性

作用：用于指定 bean 的作用范围

取值：常用的就是单例和多例

- singletond : 单例的（default）
- prototype : 多例的
- request : 作用于 web 应用的请求范围
- session : 作用于 web  应用的会话范围
- global-session : 作用于集群的会话范围（全局会话范围），当不是集群范围时，它就是 session

![](session-988affbd-bff9-48fc-bb1d-5e135fe32082.png)

(2) bean对象的声明周期

单例对象：

- 出生：当容器创建时发生
- 活着：只要容器还在对象就一直活着
- 死亡：容器销毁，对象消亡

总结：单例对象的声明周期和容器相同

多例对象：

- 出生：当我们使用对象时 Spring 框架为我们创建
- 活着：对象只要是在使用过程中就活着
- 死亡：当对象长时间不用，且没有别的对象引用时，由 Java 的GC回收

## 三、依赖注入（Dependency Injection）

### 1.概述

IOC的作用：减低程序间的耦合（即依赖关系）

在当前类需要用到其他类的对象，由 Spring 为我们提供，而我们在配置文件中说明依赖关系的维护，这种方式就称为依赖注入。

能注入的数据：

- 基本类型和 String
- 其他 bean 类型（在配置文件中或者注解中配置过的bean）
- 复杂类型/集合类型

### 2.注入方式

1.第一种：使用构造函数提供

使用的标签：constructor-arg

标签所在位置：bean 标签的内部

标签中的属性：

- type : 用于指定要注入的数据类型，该类型也是构造函数中某个或某些参数的类型
- index : 用于指定要注入的数据给构造函数中指定索引位置的参数赋值，索引的位置时从0开始
- name(常用) : 用于指定给构造函数中指定名称的参数赋值
- value : 用于提供基本类型和String类型的数据
- ref : 用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器出现过的bean对象

例：

    public class AccountServiceImpl implements IAccountService {
        // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
        private String name;
        private Integer age;
        private Date birthday;
    
        public AccountServiceImpl(String name, Integer age, Date birthday){
            this.name = name;
            this.age = age;
            this.birthday = birthday;
        }
    
        public void  saveAccount() {
            System.out.println("service中的saveaccount()执行了");
        }
    
    }

测试类：

    public static void main(String[] args) {
            //1.获取核心容器对象
            ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
            //2.根据id获取Bean对象
            IAccountService as  = (IAccountService)ac.getBean("accountService");
            as.saveAccount();
        }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <constructor-arg name = "name" value="taylor"></constructor-arg>
            <constructor-arg name = "age" value = "23"></constructor-arg>
            <constructor-arg name = "birthday" ref = "now"></constructor-arg>
        </bean>
    
        <bean id = "now" class = "java.util.Date"></bean>

特点：在获取 bean 对象时，注入数据是必须的操作，否则无法操作成功

弊端：改变了 bean 对象的实例化方式，使我们在用不到这些数据的情况下也必须提供带参构造函数，因此开发中较少使用此方法，除非避无可避

2.第二种：使用 set 方法提供（更常用的方式）

使用的标签：property

出现的位置：bean 标签的内部

标签的属性：

name : 用于指定注入时所使用的 set 方法

value : 用于提供基本类型和 String 类型的数据

ref : 用于指定其他的bean类型数据，它指的就是在 Spring 容器中出现过的bean对象

优势：创建对象时没有明确的限制，可以直接使用默认构造函数

弊端：如果有某个成员必须有值，是有可能 set 方法没有执行

(1) 基本类型和 String 的注入方式

    public class AccountServiceImpl implements IAccountService {
        // 如果时经常变化的数据不适用于依赖注入，此处仅为演示
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
    
        public void  saveAccount() {
            System.out.println("service中的saveaccount()执行了" + name + "," + age + "," +birthday);
        }
    
    }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <property name="name" value ="taylor"></property>
            <property name="age" value="21"></property>
            <property name="birthday" ref="now"></property>
        </bean>
    
        <bean id = "now" class = "java.util.Date"></bean>

测试类同上

(2) 复杂集合类型的注入方式

用于给 List 结构集合注入的标签

- list
- array
- set

用于给map结构集合注入的标签

- map
- properties

结构相同，标签可以互换，因此开发中只要记住两组标签即可

类：

    public class AccountServiceImpl implements IAccountService {
    
        private String[] myStrs;
        private List<String> myList;
        private Set<String> mySet;
        private Map<String, String> myMap;
        private Properties myProps;
    
        public void setMyStrs(String[] myStrs) {
            this.myStrs = myStrs;
        }
    
        public void setMyList(List<String> myList) {
            this.myList = myList;
        }
    
        public void setMySet(Set<String> mySet) {
            this.mySet = mySet;
        }
    
        public void setMyMap(Map<String, String> myMap) {
            this.myMap = myMap;
        }
    
        public void setMyProps(Properties myProps) {
            this.myProps = myProps;
        }
    
        public void  saveAccount() {
            System.out.println(Arrays.toString(myStrs));
            System.out.println(myList);
            System.out.println(myMap);
            System.out.println(mySet);
            System.out.println(myProps);
        }
    
    }

配置如下：

    <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl">
            <!--以下三个标签是等价的，set未列出-->
            <property name="myList">
                <list>
                    <value>aaa</value>
                    <value>bbb</value>
                </list>
            </property>
    
            <property name="myStrs">
                <array>
                    <value>aaa</value>
                    <value>bbbb</value>
                </array>
            </property>
    
            <property name="mySet">
                <array>
                    <value>aaa</value>
                    <value>bbbb</value>
                </array>
            </property>
    
            <!--以下两种方式等价-->
            <property name="myMap">
                <map>
                    <!--以下两种配置方式都可以-->
                    <entry key="testA" value="aaa"></entry>
                    <entry key="testA">
                        <value>bbb</value>
                    </entry>
                </map>
            </property>
    
            <property name="myProps">
                <props>
                    <prop key="testB">bbb</prop>
                </props>
            </property>
        </bean>

3.第三种：使用注解提供

**如何使用？**

第一步：在类或方法的前面加上注解关键字

第二步：引入约束,注意此处约束多了xmlns:context...

第三步：添加配置文件，告知 Spring 在创建容器时要扫描的包，配置所需的标签不是在 bean 约束中，而是一个名称为context 的名称孔家和约束中,完整配置如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
        <context:component-scan base-package="com.itheima"></context:component-scan>
    </beans>

**有哪些注解？**

(1)用于创建对象的

作用：等同于 xml 配置文件中编写一个 <bean> 标签

@Component

形式：@Component(value=" ")/@Component(" ")

作用：用于把当前类对象存入 Spring 容器中

属性：

value : 用于指定 bean 的 id，当我们不写的时候，它的默认值是当前类名，且首字母改小写;当值只有一个的时候可以省略

以下三个注解的作用与@Component完全一样，它们是 Spring 提供的更明确的划分，使三层对象更加清晰

- @Controller  用于表现层
- @Service       用于业务层
- @Repository 用于持久层

(2) 用于注入数据的

作用：等同于在 <bean> 标签中写一个 <property> 标签

@Autowired

作用：自动按照类型注入，只要容器中有唯一的一个 bean 对象类型和要注入的变量类型匹配，就可以注入成功如果IOC容器中没有任何 bean 的类型和要注入的变量类型匹配，则报错

出现位置：可以是变量上，也可以是方法上，

细节：在使用注解注入时，set 方法就不是必须的了

@Qualifier

作用：在按照类型注入的基础上再按照名称注入，它在给类成员注入时不能单独使用，但是在给方法参数注入          时可以。

属性：

value : 用于指定注入的 bean 的  id

@Resource

作用：直接按照 bean 的 id 注入，可以直接使用

属性：

name : 用于指定 bean 的 id

等同于@Autowired+@Qualifier

以上三个注入都只能注入其他 bean 类型的数据，而基本类型和 String 类型的数据无法使用上述注解实现。另外，集合类型的注入只能通过 xml 配置文件实现

@Value

作用：用于注入基本类型和 String 类型的数据

属性：

value : 用于指定数据的值，它可以使用 Spring 中 Spel (即spring的el表达式)

Spel 的写法：${表达式}

**操作实例：**

接口如下：

    public interface IAccountDao {
    
        void saveAccount();
    }
    
    public interface IAccountService {
    
        /**
         * 模拟保存账户
         */
        void saveAccount();
    }

实现类：

    @Service("accountService")
    public class AccountServiceImpl implements IAccountService {
    
        @Autowired
        @Qualifier("accountDao2")
        private IAccountDao accountDao = null;


​    
        public void  saveAccount() {
            accountDao.saveAccount();
        }
    
    }
    
    @Repository("accountDao1")
    public class AccountDaoImpl implements IAccountDao {
    
        public void  saveAccount() {
            System.out.println("对象创建了111");
        }
    
    }
    
    @Repository("accountDao2")
    public class IAccountDaoImpl2 implements IAccountDao{
    
        public void  saveAccount() {
            System.out.println("对象创建了222");
        }
    }

测试类：

    public static void main(String[] args) {
            //1.获取核心容器对象
            ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
            //2.根据id获取Bean对象
            IAccountService as  = (IAccountService)ac.getBean("accountService");
    
            as.saveAccount();
        }
    }

如上，AccountDaoImpl1 和 AccountDaoImpl2 实现接口 IAccountDao ，两个类中分别实现了不同的 saveAccount() 方法，AccountServiceImpl 实现接口 IAccountService ，其中调用了 IAccountDao 接口。AccountServiceImpl 通过注解关键字 Autowired 去 Spring 容器中寻找 accountDao ， 再根据 Qualifier 配置的 value 找到两个 dao 的实现类中与之相匹配的 Repository 的值。

(3) 用于改变范围的

作用：等同于在 <bean> 标签中使用 scope 属性

@Scope

作用：用于指定 bean 的作用范围

属性：

value : 指定范围的取值，同 xml 中值，常用为 singleton ,  prototype

(4) 和生命周期相关（了解）

作用：等同于在<bean>标签中使用 init-method 和 destroy-method

@PreDestory

作用：用于指定销毁方法

@Postcontrust

作用：用于指定初始化方法

## 四、基于 XML 的 IOC 案例

1. 创建数据库

        create table account(
        	id int primary key auto_increment,
        	name varchar(40),
        	money float
        )character set utf8 collate utf8_general_ci;
        
        insert into account(name,money) values('aaa',1000);
        insert into account(name,money) values('bbb',1000);
        insert into account(name,money) values('ccc',1000);

2.  文件结构如下

    ![](Untitled-b8f378d2-1d05-4829-918a-6674b2da7ca7.png)

3. 创建接口
    - IAccountDao

            public interface IAccountDao {
            
                /**
                 * 查询所有
                 * @return
                 */
                List<Account> findAllAccount();
            
                /**
                 * 查询一个
                 * @return
                 */
                Account findAccountById(Integer accountId);
            
                /**
                 * 保存
                 * @param account
                 */
                void saveAccount(Account account);
            
                /**
                 * 更新
                 * @param account
                 */
                void updateAccount(Account account);
            
                /**
                 * 删除
                 * @param accountId
                 */
                void deleteAccount(Integer accountId);
            }

    - IAccountService

            public interface IAccountService {
            
                /**
                 * 查询所有
                 * @return
                 */
                List<Account> findAllAccount();
            
                /**
                 * 查询一个
                 * @return
                 */
                Account findAccountById(Integer accountId);
            
                /**
                 * 保存
                 * @param account
                 */
                void saveAccount(Account account);
            
                /**
                 * 更新
                 * @param account
                 */
                void updateAccount(Account account);
            
                /**
                 * 删除
                 * @param accountId
                 */
                void deleteAccount(Integer accountId);
            }

4. 创建实现类
    - AccountDaoImpl

            public class AccountDaoImpl implements IAccountDao {
            
                private QueryRunner runner;
            
                public void setRunner(QueryRunner runner) {
                    this.runner = runner;
                }
            
                public List<Account> findAllAccount() {
                    try{
                        return runner.query("select * from account", new BeanListHandler<Account>(Account.class));
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            
                public Account findAccountById(Integer accountId) {
                    try{
                        return runner.query("select * from account where id = ?", new BeanHandler<Account>(Account.class),accountId);
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            
                public void saveAccount(Account account) {
                    try{
                        runner.update("insert into account(name, money) values(?,?)", account.getName(),account.getMoney());
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            
                public void updateAccount(Account account) {
                    try{
                        runner.update("update account set name = ?, money = ? where id = ?", account.getName(),account.getMoney(),account.getId());
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            
                public void deleteAccount(Integer accountId) {
                    try{
                        runner.update("delete from account where id = ?", accountId);
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }

    - AccountServiceImpl

            public class AccountServiceImpl implements IAccountService {
            
                private IAccountDao accountDao;
            
                public void setAccountDao(IAccountDao accountDao) {
                    this.accountDao = accountDao;
                }
            
                public List<Account> findAllAccount() {
                    return accountDao.findAllAccount();
                }
            
                public Account findAccountById(Integer accountId) {
                    return accountDao.findAccountById(accountId);
                }
            
                public void saveAccount(Account account) {
                    accountDao.saveAccount(account);
                }
            
                public void updateAccount(Account account) {
                    accountDao.updateAccount(account);
                }
            
                public void deleteAccount(Integer accountId) {
                    accountDao.deleteAccount(accountId);
                }
            }

5.  创建账户实体类

- Account

        public class Account implements Serializable {
        
            private Integer id;
            private String name;
            private Float money;
        
            public void setId(Integer id) {
                this.id = id;
            }
        
            public void setName(String name) {
                this.name = name;
            }
        
            public void setMoney(Float money) {
                this.money = money;
            }
        
            public Integer getId() {
                return id;
            }
        
            public String getName() {
                return name;
            }
        
            public Float getMoney() {
                return money;
            }
        
            @Override
            public String toString() {
                return "Account{" +
                        "id=" + id +
                        ", name='" + name + '\'' +
                        ", money=" + money +
                        '}';
            }
        }

6. 创建测试类

- AccountServiceTest

        public class AccountServiceTest {
        
            @Test
            public void testFindAll() {
                // 1.获取容器
                ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                // 2.得到业务层对象
                IAccountService iAccountService = applicationContext.getBean("accountService",IAccountService.class);
                // 3.执行方法
                List<Account> accounts = iAccountService.findAllAccount();
                for (Account account : accounts) {
                    System.out.println(account);
                }
            }
        
            @Test
            public void testFindOne() {
                // 1.获取容器
                ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                // 2.得到业务层对象
                IAccountService iAccountService = applicationContext.getBean("accountService",IAccountService.class);
                // 3.执行方法
                Account account = iAccountService.findAccountById(1);
                System.out.println(account);
            }
        
            @Test
            public void testSave() {
                Account account = new Account();
                account.setName("test");
                account.setMoney(12345f);
                // 1.获取容器
                ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                // 2.得到业务层对象
                IAccountService iAccountService = applicationContext.getBean("accountService",IAccountService.class);
                // 3.执行方法
                iAccountService.saveAccount(account);
            }
        
            @Test
            public void testUpdate() {
                // 1.获取容器
                ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                // 2.得到业务层对象
                IAccountService iAccountService = applicationContext.getBean("accountService",IAccountService.class);
                // 3.执行方法
                Account account = iAccountService.findAccountById(4);
                account.setMoney(23456f);
                iAccountService.updateAccount(account);
            }
        
            @Test
            public void testDelete() {
                // 1.获取容器
                ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
                // 2.得到业务层对象
                IAccountService iAccountService = applicationContext.getBean("accountService",IAccountService.class);
                // 3.执行方法
                iAccountService.deleteAccount(4);
            }
        }

7. 配置 bean.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--配置Service-->
        <bean id="accountService" class="com.greyson.service.impl.AccountServiceImpl">
            <!--注入dao-->
            <property name="accountDao" ref="accountDao"></property>
        </bean>
    
        <!--配置dao-->
        <bean id="accountDao" class="com.greyson.dao.impl.AccountDaoImpl">
            <!--注入runner-->
            <property name="runner" ref="runner"></property>
        </bean>
        
        <!--配置runner-->
        <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
            <!--注入数据源-->
            <constructor-arg name="ds" ref="dataSource"></constructor-arg>
        </bean>
        
        <!--配置数据源-->
        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
            <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
            <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/eesy"></property>
            <property name="user" value="root"></property>
            <property name="password" value="HotteMYSQL"></property>
        </bean>
    </beans>

## 五、基于注解的IOC实例

代码重复太多，就不贴出来了，主要是对于注解的应用，建议也手动敲一遍代码，对记忆和理解的加深有帮助。

1. Spring 中的新注解：

    @Configuration

    作用：指定当前类是一个配置类

    细节：当配置类作为 AnnotationConfigurationApplicationContext 对象创建的参数时，该注解可以不写

    @ComponentScan

    作用：用于通过注解指定 Spring 在创建容器时要扫描的包

    属性：

    value : 它和 basepackages 的作用是一样的，都是用于指定创建容器时要扫描的包

    使用此注解就等同于在 xml 中配置了：

    <context:component-scan base-package="com.greyson"></context:component-scan>

    @Bean

    作用：用于把当前方法的返回值作为 bean 对象放入 Spring 的IOC容器中

    属性：

    name : 用于指定 bean 的 id，当不写时，默认值为当前方法的名称

    细节：

    当我们使用注解配置方法时，如果方法有参数，Spring 框架会去容器中查找有没有可用的 bean 对象，

    查找的方式和 Autowired 注解的作用是一样的

    @I*mport*

    作用：用于导入其他的配置类

    属性：

    value : 用于指定其他配置类的字节码

    当我们使用 Import 的注解之后，有 Import 注解的类就是父配置类，而导入的都是子配置类

    @Properties

    作用：用于指定 properties 文件的位置

    属性：

    value : 指定文件的名称和路径

    关键字：classpath , 表示类路径下

## 六、Spring 整合 Junit（后续补充）

Spring 整合 Junit 的配置过程：

1. 导入 Spring 整合 Junit 的 jar ( 坐标 )
2. 使用 Junit 提供的一个注解把原有的 main 方法替换了，替换成 Spring 提供的

    @Runwith

3. 告知 Spring 的运行器， Spring 和 ioc 创建是基于 xml 还是注解的，并且说明位置，用到的注解如下

    @ContextConfiguration

    Locations : 指定 xml 文件的位置，加上 classpath 关键字，表示在类路径下

    classes : 指定注解类所在地位置

4. 使用@Autowired 给测试类中的变量注入数据

## 七、事务管理（后续补充）

事务：无法被分割的操作。以银行转账业务来说，转账的具体步骤是：转出人的账户资金减少，同等资金被增加到被转账者的账户上，如果转出人的账户资金减少了，而因某些因素导致被转账者的账户资金增加操作失败，那么转账业务将回滚到初始状态，包括撤销对转出人账户资金减少这个已经成功的操作。

## 八、动态代理分析（后续补充）

什么是代理？

![](-d7c45db4-1f69-4b5c-b5f5-a9624176e6f8.png)

以上需要后续补充的都是我掌握的不好的部分

## 九、AOP( 面向切面编程 )

AOP采取的是横向抽取机制，取代了传统纵向继承体系重复性代码。

### 那么何为软件的横向和纵向？

从纵向结构来看就是我们软件的各个模块，它所负责的是软件的核心业务（如购商品购买、添加购物车等）；从横向来看的话，软件的各个模块之间又有所关联，其中会包含一些公共模块（例如日志、权限等）；这些公共模块可以存在于各个核心业务中，而AOP的处理将两者分离，使开发人员可以专注于核心业务的开发，提高了开发效率。

我们知道在单体架构下的软件开发中，一个大型项目通常是依照功能拆分成各个模块。但是如日志、安全和事务管理此类重要且繁琐的开发却没有必要参与到各个模块中，将这些功能与业务逻辑相关的模块分离就是面向切面编程所要解决的问题

### AOP底层原理

使用动态代理实现
（1）基于JDK的代理

    适用于有接口情况，使用动态代理创建接口实现类代理对象

（2）基于CGLIB动态代理

    适用于没有接口情况，使用动态代理创建类的子类代理对象