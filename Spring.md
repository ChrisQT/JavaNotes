 

# 一、SpringFramework系统架构

- 单元测试与集成测试
- Core Container：核心容器
- Aspects：AOP思想实现
- AOP:面向切向编程
- Web开发
- Data Access：数据访问
- Data Integration：数据集成



# 二、核心概念

- IoC/DI

  - 代码书写现状：耦合度高

  ```java
  public class BookServiceImpl implements BookService{
      private BookDao bookDao = new BookDaoImpl();
      public void save(){
          bookDao.save();
      }
  }
  ```

  ```java
  public class BookDaoImpl implements BookDao{
      public void save(){
          System.out.println("save..")
      }
  }
  ```

  ```java
  # 底层改变，更换实现方式
  public class BookDaoImpl2 implements BookDao{
      public void save(){
          System.out.println("save2..")
      }
  }
  ```

  由BookDaoImpl改成BookDaoImpl2后，``private BookDao bookDao = new BookDaoImpl();``也需要改变

  - 解决方案：使用对象时，程序不主动使用new生成对象，转为外部提供对象
  - IoC(Inversion of Controll)控制反转：对象的创建控制权由程序转移到外部
  - Spring技术对IoC思想进行了实现
    - Spring提供了一个容器，成为IoC容器 ，充当IoC思想中的外部容器
    - IoC容器负责对象的创建，初始化等一系列工作，被创建或管理的对象称为Bean

- DI(Dependency Injection) 依赖注入
  - 在容器中建立bean与bean的依赖关系的整个过程，成为依赖注入

    Service对象 ->(依赖) Dao对象

- 目标：充分解耦
  - 使用IoC容器管理Bean
  - 在IoC容器中将有依赖关系的Bean进行绑定

# 三、IoC入门案例

1. 管理什么？（Service和Dao）
2. 如何将被管理的对象告知IoC容器（配置）
3. 被管理的对象交给IoC容器，如何获取IoC容器？（接口）
4. IoC容器得到后，如何从容器中获取Bean？（接口方法）
5. 使用Spring导入哪些坐标？（pom.xml）

- 配置pom.xml

```html
<dependencies>
    <dependency>
       <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

- 配置bean

  resource下新建applicationContext.xml，配置bean信息

  ```html
  <bean id="BookDao" class="com.xjtu.dao.impl.BookDaoImpl"/>
  <bean id="BookService" class="com.xjtu.service.impl.BookServiceImpl"/>
  ```

  id：给bean取名字

  class：定义bean类型

- 获取IoC容器

  在App启动类中获取IoC对象

- 获取bean

```java
public static void main(String[] args) {
//    BookService bookService = new BookServiceImpl();
//    bookService.save();
    // get IoC container
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    // get Bean
//    BookDao bookDao = (BookDao)ctx.getBean("bookDao");
//    bookDao.save();
    BookService bookService = (BookService)ctx.getBean("bookService");
    bookService.save();
}
```

# 四、DI入门案例

1. 基于IoC管理bean
2. Service中使用new形式创建的Dao对象是否保留？（否）
3. Service中需要的Dao对象如何进入到Service中？（提供方法）
4. Service和Dao之间的关系如何描述？（配置）

- 删除ServiceImpl中new Dao对象的语句，提供对应的set方法（IoC容器自动执行）。

```java
public class BookServiceImpl implements BookService {
    private BookDao bookDao;

    public void save() {
        System.out.println("Book Service Saved!");
        bookDao.save();
    }

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

- 修改xml，配置Service与dao的关系
  - property表示配置当前bean的属性
  - name表示配置哪一个具体的属性，对应：``private BookDao bookDao;``
  - ref表示参照哪一个bean，对应：``<bean id="bookDao"  class="..."/>``

```html
<bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl"/>
<bean id="bookService" class="com.xjtu.service.impl.BookServiceImpl">
    <property name="bookDao" ref="bookDao"/>
</bean>
```

# 五、Bean的配置

- Bean可以起多个别名，用name指定，用空格/逗号/分号分隔

```html
<bean id="bookDao" name="bookDao2" class="com.xjtu.dao.impl.BookDaoImpl"/>
```

- Spring默认创建的Bean是单例

```html
<!--默认singleton单例 -->
<bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl" scope="singleton"/>
<!-- 非单例 -->
<bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl" scope="prototype"/>
```

- 适合交给容器进行管理的对象：
  - 表现层对象（Servlet）
  - 业务层对象（Service）
  - 数据层对象（Dao）
  - 工具对象
- 不适合交给容器管理的对象：
  
- 封装实体的域对象（有状态）
  
- Bean实例化

  Spring构造Bean的时候调用的是无参的构造方法

# 六、使用静态工厂实例化Bean(较少使用)

```java
public class OrderDaoFactory{
    public static OrderDao getOrderDao(){
        return new OrderDaoImpl();
    }
}
```

```java
<bean id="orderDao" class="com.xjtu.dao.factory.OrderDaoFactory" factory-method="getOrderDao"/>
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    OrderDao orderDao = (OrderDao)ctx.getBean("orderDao");
    orderDao.save();
}
```

- 为什么需要使用静态工厂实例化Bean

  一般工厂模式出厂之前需要完成一定的配置，使用new创建或者使用IoC获取都可能无法完成初始化步骤。

# 七、实例工厂初始化Bean

- 方法一：

```java
public class UserDaoFactory {
    public UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

```html
<!--先造factory bean-->
<bean id="userFactory" class="com.xjtu.dao.factory.UserDaoFactory"/>
<!--再造dao bean指向factory bean-->
<bean id="userDao" factory-bean="userFactory" factory-method="getUserDao"/>
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserDao userDao = (UserDao)ctx.getBean("userDao");
    userDao.save();
}
```

- 方法二（较常用）

```java
public class UserDaoFactoryBean implements FactoryBean {
    public Object getObject() throws Exception {
        return new UserDaoImpl();
    }

    public Class<?> getObjectType() {
        return UserDao.class;
    }
    
    public boolean isSingleton() {
        return false;
    }
}
```

```html
<bean id="userDao" class="com.xjtu.dao.factory.UserDaoFactoryBean"/>
```

# 八、Bean生命周期

- 生命周期：从创建到消亡的完整过程
- bean生命周期控制：在bean创建后销毁前的操作

```java
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println("Book Dao Saved!");
    }

    public void init(){
        System.out.println("book dao init!");
    }

    public void destory(){
        System.out.println("book dao destroy!");
    }
}

```

```html
<bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl" init-method="init" destroy-method="destory"/>
```

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    BookService bookService = (BookService)ctx.getBean("bookService");
    bookService.save();
    ctx.close();
}
```

代码绑定bean对象的初始化操作和销毁操作。销毁操作必须手动关闭容器才能看到。

#或者使用关闭钩子

```java
ctx.registerShutdownHook();
```

或者使用接口的方式实现，__无需再进行配置xml__:

```java
public class BookServiceImpl implements BookService, InitializingBean, DisposableBean {
    private BookDao bookDao;

    public void save() {
        System.out.println("Book Service Saved!");
        bookDao.save();
    }

    public void setBookDao(BookDao bookDao) {
        System.out.println("set book dao!");
        this.bookDao = bookDao;
    }

    public void destroy() throws Exception {
        System.out.println("book service destroy!");
    }
	// 属性设置之后
    public void afterPropertiesSet() throws Exception {
        System.out.println("book service after property set");
    }
```

- Bean生命周期
  - 初始化容器
    - 创建对象（内存分配）
    - 执行构造方法
    - 执行属性注入（set操作）
    - 执行bean初始化方法
  - 使用bean
    - 执行业务操作
  - 关闭/销毁bean
    - 执行bean销毁方法

- Bean销毁时机

  - 容器关闭前出发bean的销毁

  - 关闭容器方式

    - 手工关闭容器

    ``ConfigurableApplicationContext``接口执行``close()``操作

    - 注册关闭钩子，再虚拟机退出前先关闭容器再退出虚拟机

    ``ConfigurableApplicationContext``接口执行``registerShutdownHook``操作

# 九、依赖注入方式

- 像一个类中传递数据的方式有几种？

  - 普通方法
  - 构造方法

- 依赖注入描述了容器中建立bean和bean依赖关系的过程，如果bean运行所需的是数字或者字符串如何进行依赖注入？

  - 引用类型
  - 简单类型（基本数据类型与String）

- 依赖注入方式

  - setter注入（略）
    - 简单类型（同下）
    - 引用类型*（已介绍）

   ```java
  public class BookDaoImpl implements BookDao {
      private int connectionNum;
      private String databaseName;
  
      public void save() {
          System.out.println("book dao saved in " + databaseName + connectionNum);
      }
  	// 省略若干函数
      public void setConnectionNum(int connectionNum) {
          this.connectionNum = connectionNum;
      }
  
      public void setDatabaseName(String databaseName) {
          this.databaseName = databaseName;
      }
  }
   ```

  ```html
  <bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl">
      <property name="connectionNum" value="10"/>
      <property name="databaseName" value="mysql"/>
  </bean>
  ```

  - 构造器注入
    - 引用类型
    - 简单类型

  ```java
  public class BookServiceImpl2 implements BookService {
      private BookDao bookDao;
      private String description;
  
      public BookServiceImpl2(BookDao bookDao, String description) {
          this.bookDao = bookDao;
          this.description = description;
      }
  
      public void save() {
          System.out.println("Book Serivce2 save" + description);
      }
  }
  ```

  ```html
  <bean id="bookService2" class="com.xjtu.service.impl.BookServiceImpl2">
      <constructor-arg name="bookDao" ref="bookDao"/>
      <constructor-arg name="description" value="haha"/>
  </bean>
  ```

  xml配置也可以这么写，与形参名解耦：

  ```html
  <bean id="bookService2" class="com.xjtu.service.impl.BookServiceImpl2">
      <constructor-arg type="com.xjtu.dao.BookDao" ref="bookDao"/>
      <constructor-arg type="java.lang.String" value="haha"/>
  </bean>
  ```

  ```html
  <bean id="bookService2" class="com.xjtu.service.impl.BookServiceImpl2">
      <constructor-arg index="0" ref="bookDao"/>
      <constructor-arg index="1" value="haha"/>
  </bean>
  ```

- 依赖注入方式选择

  - 强制依赖使用构造器进行，使用setter注入有概率不进行注入导致null值出现
  - 可选依赖使用setter进行，灵活性强
  - Spring框架倡导使用构造器注入，第三方框架大多使用构造器注入
  - 自己开发的模块推荐使用setter注入

# 十、依赖自动装配

- IoC容器根据bean所依赖的资源在容器中自动查找并注入到bean中的过程称为自动装配

- 自动装配方法

  - 按类型（常用）

    自动完成了bookService和bookDao的Bean对象的绑定（前提是bookService里面必须有bookDao对象的set方法，且IoC容器中有且仅有一个bookDao的Bean对象）

  ```html
  <bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl"/>
  <bean id="bookService" class="com.xjtu.service.impl.BookServiceImpl" autowire="byType"/>
  ```

  - 按名称

    前提是set方法名除去set第一个字母变小写的部分，与bean id完全一致。

  ```html
  <bean id="bookDao" class="com.xjtu.dao.impl.BookDaoImpl"/>
  <bean id="bookService" class="com.xjtu.service.impl.BookServiceImpl" autowire="byName"/>
  ```

  - 按构造方法（不推荐）

- 依赖自动装配特征
  - 自动装配用于引用类型的依赖自动注入，不能对简单类型进行操作
  - 使用按类型装配时（byType）必须保证容器内相同类型的bean唯一，推荐使用
  - 使用按名称装配时（byName）必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐使用
  - 自动装配优先级低于setter注入和构造器注入，同时出现时自动装配配置失效

# 十一、集合注入

集合：数组、List、Set、Map、Properties

```java
public class BooksDaoImpl implements BookDao {
    private int[] array;
    private List<String> list;
    private Set<String> set;
    private Map<String, String> map;
    private Properties properties;

    public void save() {
        System.out.println("books dao save~");
        System.out.println("数组：" + Arrays.toString(array));
        System.out.println("List：" + list);
        System.out.println("Set：" + set);
        System.out.println("Map：" + map);
        System.out.println("Properties：" + properties);
    }

    public void setArray(int[] array) {
        this.array = array;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void setProperties(Properties properties) {
        this.properties = properties;
    }
}
```

```html
    <bean id="bookDao" class="com.xjtu.dao.impl.BooksDaoImpl">
        <property name="array">
            <array>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </array>
        </property>

        <property name="list">
            <list>
                <value>a</value>
                <value>b</value>
                <value>c</value>
            </list>
        </property>

        <property name="set">
            <set>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </set>
        </property>

        <property name="map">
            <map>
                <entry key="1" value="a"/>
                <entry key="2" value="b"/>
                <entry key="3" value="c"/>
            </map>
        </property>

        <property name="properties">
            <props>
                <prop key="a">x</prop>
                <prop key="b">y</prop>
                <prop key="c">z</prop>
            </props>
        </property>
    </bean>

    <bean id="bookService" class="com.xjtu.service.impl.BookServiceImpl" autowire="byType"/>
```

*<array>和<list>可以混用

*如果是引用类，<value>改成<ref bean="">即可。

# 十二、数据源对象管理

- 在pom文件中导入druid包

```html
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.38</version>
</dependency>
```

- 在applicationContext.xml中配置DruidDataSource的Bean

```html
<!--管理DruidDataSource对象-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/employees"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
```

- 在main方法中获取Bean

```java
public static void main(String[] args) throws SQLException {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    DataSource dataSource = (DataSource) ctx.getBean("dataSource");
    Connection conn = dataSource.getConnection();
    String sql = "select first_name from employees limit 1";
    Statement stmt = conn.createStatement();
    ResultSet res = stmt.executeQuery(sql);
    res.next();
    System.out.println(res.getString(1));
}
```

# 十三、加载properties配置信息

0. resource文件夹下，新建jdbc.properties文件

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/employees
jdbc.username=root
jdbc.password=123456
```

1. 开启context命名空间

```html
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
        ">
	<!--省略中间-->
</beans>
```

2. 在applicationContext.xml中加载properties文件

```html
    <context:property-placeholder location="jdbc.properties"/>
```

3. 使用属性占位符读取properties文件中的属性

```html
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

*如果属性名过于常用，可能与系统环境变量冲突，需要避免覆盖

```html
    <context:property-placeholder location="jdbc.properties" system-properties-mode="NEVER"/>
```

*如果需同时加载多个配置文件

```html
    <context:property-placeholder location="jdbc.properties, other.properties"/>
```

或者：

```html
    <context:property-placeholder location="*.properties"/>
```

或者自动扫描开发目录下的properties文件：

```html
    <context:property-placeholder location="classpath:*.properties/>
```

甚至包含其他jar包的properties文件（推荐）：

```html
    <context:property-placeholder location="classpath*:*.properties/>
```

# 十四、容器

- 配置文件不在工程目录下，在文件系统中时：

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("absolute/path/to/applicationContext.xml")
```

- 获取bean后类型强转的替代方式：

```java
BookDao bookDao = ctx.getBean("bookDao", BookDao.class);
// 或者容器中只有一个BookDao类型的bean时
BookDao bookDao = ctx.getBean(BookDao.class);
```

- 加载多个配置文件（相当于合并）

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("bean1.xml", "beans.xml")
```

# 十五、总结

- bean设置相关

| <bean                                          |                                     |
| ---------------------------------------------- | ----------------------------------- |
| id="bookDao"                                   | bean的 id                           |
| name="dao daoImpl"                             | bean别名                            |
| class="com.xjtu.dao.impl.BookDaoImpl"          | bean类型，静态工厂类，FactoryBean类 |
| scope="singleton"                              | 控制bean的实力数量                  |
| init-method="init"                             | 生命周期初始化方法                  |
| destory-method="destory"                       | 生命周期销毁方法                    |
| autowire="byType"                              | 自动装配类型                        |
| factory-method="getInstance"                   | bean工厂方法                        |
| factory-bean="com.xjtu.factory.BookDaoFactory" | 实例化工厂bean                      |
| lazy-init="true"                               | 控制bean延迟加载                    |
| />                                             |                                     |

- 依赖注入相关

| <bean id="bookService" class="com.xjtu.dao.impl.BookSerivecImpl"> |                    |
| ------------------------------------------------------------ | ------------------ |
| <constructor-arg name="bookDao" ref="bookDao"/>              | 构造器注入应用类型 |
| <constructor-arg name="userDao" ref="userDao"/>              |                    |
| <constructor-arg name="msg" value="WARN"/>                   | 构造器注入简单类型 |
| <constructor-arg type="java.lang.String" index="3" value="WARN"/> |                    |
| <property name="bookDao" ref="bookDao"/>                     | setter注入引用类型 |
| <property name="userDao" ref="userDao"/>                     |                    |
| <property name="msg" ref="WARN"/>                            | setter注入简单类型 |
| <property name="names">                                      | list集合           |
| <list>                                                       |                    |
| <value>apple</value>                                         | 集合注入简单类型   |
| <ref bean="dataSource">                                      | 集合注入应用类型   |
| </list>                                                      |                    |
| </property>                                                  |                    |
| </bean>                                                      |                    |

# 十六、注解开发

1. 在需要配置的Bean的类别定义代码前一行进行注解配置

```java
@Component("bookDao")
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println("book dao saved!");
    }
}
```

2. 在applicationContext.xml中进行配置，使IoC容器找到注解并创建bean

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
        ">
    <context:component-scan base-package="com.xjtu"/>
</beans>
```

3. 在main方法中获取bean

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    BookDao bookDao = (BookDao) ctx.getBean("bookDao");
    System.out.println(bookDao);
}
```

- 也可以不指定bean名称

```java
@Component
public class BookServiceImpl implements BookService {
    private BookDao bookDao;

    public void save() {
        System.out.println("Book Service Saved!");
        bookDao.save();
    }

    public void setBookDao(BookDao bookDao) {
        System.out.println("set book dao!");
        this.bookDao = bookDao;
    }
}
```

```java
BookService bookservice = (BookService) ctx.getBean(BookService.class);
```

- Spring提供@Compoent注解的三个衍生注解
  - @Controller：用于表现层bean定义
  - @Service：用于业务层bean定义
  - @Repository：用于数据层bean定义
- 控制bean的生命周期
  - @Scope("prototype")/@Scope("singleton")
  - @PostContruct/@PreDestroy
  - 非单例时destory方法不会被执行

```java
@Repository("bookDao")
@Scope("singleton")
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println("book dao saved!");
    }

    @PostConstruct
    public void init(){
        System.out.println("book dao init!");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("book dao destroy!");
    }
}
```

# 十七、纯注解开发

核心思想：用配置类替代配置文件

- Spring3.0开启了纯注解开发模式，使用Java类替代配置文件，开启了Spring快速开发赛道
- Java类代替Spring核心配置文件

1. 写配置类

```java
@Configuration
@ComponentScan("com.xjtu")
@Import(JdbcConfig.class)
public class SpringConfig {

}
```

*@ComponentScan只能用一次，如需加载多个配置时：

```java
@ComponentScan({"com.xjtu", "com.other"})
```

2. 在main方法中获取ApplicationContext

```java
public class AppForAnnotation {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookService bookService = (BookService)ctx.getBean(BookService.class);
        System.out.println(bookService);
        bookService.save();
        DataSource ds = ctx.getBean(DataSource.class);
        System.out.println(ds);
        ctx.close();
    }
}
```

# 十八、注解开发依赖注入

- bean中的变量实现自动注入

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println("book dao saved!");
    }

    @PostConstruct
    public void init(){
        System.out.println("book dao init!");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("book dao destroy!");
    }
}
```

```java
@Component
public class BookServiceImpl implements BookService {

    @Autowired // 按类型自动装配，IoC容器可以有多个该类bean但不推荐
//    @Qualifier("bookDao2")  // 按名称注入，@Qualifier必须绑定@Autowired使用
    private BookDao bookDao;
    @Value("${name}") // name来自jdbc.property，需要在配置类中引入，如下
    private String name;

    public void save() {
        System.out.println("Book Service Saved! " + name);
        bookDao.save();
    }
}
```

```java
@Configuration
@ComponentScan("com.xjtu")
@PropertySource("jdbc.properties")  // 多个用数组，不支持通配符！
@Import(JdbcConfig.class)
public class SpringConfig {

}
```

- 注意：自动装配基于反射设计创建对象，并暴力反射对应属性为私有属性初始化数据，因此无需提供setter方法。
- 注意：自动装配建议使用无参构造方法创建对象（默认），如果不提供对应构造方法，请提供唯一的构造方法。



- 加载properties文件
  - 使用@PropertySource注解加载properties文件
  - 路径仅支持单一文件配置，多文件配置需要使用数组格式，不允许使用通配符*

# 十九、第三方bean管理&依赖注入

- 在maven中导入相关的包
- 在配置类中配置相关信息（__不推荐__）

```java
@Configuration
public class SpringConfig {
    // 添加@Bean，表示当前方法的返回值是一个Bean
    @Bean
	public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/employees");
        ds.setUserName("root");
        ds.setPassword("123456");
        return ds;
    }
}
```

- 创建新的class文件管理第三方bean，并导入配置类中（__推荐__）

```java
public class JdbcConfig {
    @Value("com.mysql.jdbc.Driver")
    private String driver;
    @Value("jdbc:mysql://localhost:3306/employees")
    private String url;
    @Value("root") // 简单类型的值可以直接给出或加载properties文件获取
    private String username;
    @Value("123456")
    private String password;

    // 引用类型的可以使用bean自动装配
    // 此处由于Spring检测到该方法输出为bean
    // 因此能无需@AutoWried自动检测装配所需BookDao bean
    @Bean
    public DataSource dataSource(BookDao bookDao){
        System.out.println(bookDao);
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}

```

```java
import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.*;

import javax.sql.DataSource;

@Configuration
@ComponentScan("com.xjtu")
@Import(JdbcConfig.class)
public class SpringConfig {
}
```

# 二十、XML配置与注解配置比较

| 功能           | XML配置                                   | 注解                                                         |
| -------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 定义bean       | bean标签                                  | @Component（@Controller、@Service、@Repository）@ComponentScan |
| 设置依赖注入   | setter注入、构造器注入、自动装配          | @AutoWired、@Qualifier、@Value                               |
| 配置第三方bean | bean标签、静态工厂、实例工厂、FactoryBean | @Bean                                                        |
| 作用范围       | Scope属性                                 | @Scope                                                       |
| 生命周期       | 标准接口（init-method, destroy-method）   | @PostConstructor、@PreDestroy                                |

# 二十一、Spring整合MyBatis

[Demo的Github链接](https://github.com/ChrisQT/Spring-MyBatis-Anno)

1. POM导入依赖包

```html
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
```

2. 创建jdbc.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///employees_copy?userSSL=false
jdbc.username=root
jdbc.password=123456
```

3. 创建需要的bean对象

```java
@Configuration
@ComponentScan("com.example")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class, MyBatisConfig.class})
public class SpringConfig {

}
```

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

```java
public class MyBatisConfig {
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasesPackage("com.example.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }

    @Bean
    MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.example.dao");
        return msc;
    }
}
```

4. 创建main方法

```java
public class App2 {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        EmployeeService employeeService = ctx.getBean(EmployeeService.class);
        Employee e = employeeService.getById(10087);
        System.out.println(e);
    }
}
```

# 二十二、Spring整合Junit

1. POM文件导入Junit

```html
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.10.RELEASE</version>
    <scope>test</scope>
</dependency>
```

2. 使用Spring整合Junit专用的类加载器

   - ``@Runwith(SpringJUnit4ClassRunner.class) `` // 固定

   - ``@ContextConfiguration(classes = config.class)`` // 配置类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class EmployeeServiceTest {
    @Autowired
    private EmployeeService employeeService;

    @Test
    public void testSelectById(){
        System.out.println(employeeService.getById(10086));
    }
}
```

# 二十三、Spring AOP

- AOP（Aspect Oriented Programming）面向切向编程，一种编程范式，指导开发者如何组织程序结构
- 作用：在不惊动原始设计的基础上为其进行功能增强
- Spring理念：无侵入式



- 连接点（JoinPoint）：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等
  - 在SpringAOP中，理解为方法的执行

- 切入点（PointCut）：匹配连接点的式子
  - 在SpringAOP中，一个切入点可以只描述一个具体方法，也可以匹配多个方法
    - 一个具体方法：com.example.dao包下的EmployeeDao接口中的无形参无返回值的save方法
    - 匹配多个方法：所有的save方法，所有的get开头的方法，所有以Dao结尾的接口中的任意方法，所有带一个参数的方法
- 通知（Advice）：在切点处执行的操作，也是共性功能
  - 在SpringAOP中，功能最终以方法的形式呈现
- 通知类：定义通知的类
- 切面（Aspect）：描述通知和切入点的对应关系