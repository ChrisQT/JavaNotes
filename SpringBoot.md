# 一、入门案例

- 新建空工程
- Project Structure -> Project Settings -> Modules -> + ->Spring Initializer -> 选择SDK填写工程信息 -> 选择SpringBoot版本 -> 勾选Spring Web
- 若spring网站的访问速度低下，可以选择阿里镜像源：https://start.aliyun.com，工程模板略有不同

### 1.1 Spring程序
```xml
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

```java
public class ServletConfig extends AbstactAnnotationConfigDispatcherServletInitializer{
    protected Class<?>[] getRootConfigClasses(){
        return new Class[]{SpringConfig.class};
    }
    protected Class<?> getServletConfigClasses(){
        return new Class[]{SpringMvcConfig.class};
    }
    protected String[] getServletMappings(){
        return new String[]{"/"};
    }
}
```

```java
@Configuration
@ComponentScan("com.example.controller")
@EnableWebMvc
public class SpringMvcConfig{
    
}
```

```java
@RestController
@RequestMapping("/books")
public class BookController{
    @Autowired
    private BookService bookService;
    @GetMapping("/{id}")
    public Result getById(@PathVariable Integer id){
        Book book = bookService.getById(id);
        Integer code = book != null ? Code.GET_OK : Code.GET_ERR;
        String msg = book != null ? "" : "数据查询失败，请重试！";
        return new Result(code, book, msg);
    }
}
```
### 1.2 SpringBoot程序
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springboot-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-demo</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

```java
@RestController
@RequestMapping("/books")
public class BookController {
    @RequestMapping("/{id}")
    public String getById(@PathVariable Integer id){
        System.out.println("id==>" + id);
        return Integer.toString(id);
    }
}
```

```java
@SpringBootApplication
public class SpringbootDemoApplication { 
    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }
}
```

# 二、SpringBoot简介

为了简化Spring应用的初始搭建和开发过程

- Spring程序缺点

  - 依赖设置繁琐

  - 配置繁琐

- SpringBoot有程序优点

  - 起步依赖（简化依赖配置）
  
    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>	
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
         </dependency>
    </dependencies>
    ```
  
  - 自动配置（简化常用工程相关配置）
  
  - 辅助功能（内置服务器）
  
    ```java
    @SpringBootApplication
    public class SpringbootDemoApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(SpringbootDemoApplication.class, args);
        }
    }
    ```
  
    SpringBoot在创建项目时，采用jar的打包方式。SpringBoot的引导类是项目的入口，运行main方法就可以启动项目
### 2.1 入门案例解析

  - starter

    SpringBoot常见项目名称，定义了当前项目使用的所有依赖坐标，以达到减小配置以来的的目的

  - parent

    所有SpringBoot项目要继承的项目，定义了若干个版本坐标号（依赖管理，而非依赖），以达到减少使用依赖冲突的目的

    spring-boot-starter-parent各版本之间存在着诸多坐标版本不同，使用坐标时，仅书写G(roupid) A(rtifactid) V(ersion)的GA，V由SpringBoot提供，除非SpringBoot未提供对应版本。

  - 引导类

    - SpringBoot的引导类是Boot工程的执行入口，运行main方法就可以启动项目
    - SpringBoot工程运行后初始化Spring容器，扫描引导类所在包加载bean
    
  - 内嵌Tomcat

### 2.2 Spring与SpringBoot程序对比

|      类/配置文件       |  Spring  | SpringBoot |
| :--------------------: | :------: | :--------: |
|    pom文件中的坐标     | 手工添加 |  勾选添加  |
|      web3.0配置类      | 手工制作 |     无     |
| Spring/SpringMVC配置类 | 手工制作 |     无     |
|         控制器         | 手工制作 |  手工制作  |

- 基于idea开发SpringBoot程序需要确保联网且能够加载到程序框架结构

### 2.3 SpringBoot项目快速启动

1. 对SpringBoot项目打包（执行maven构建指令package）

2. 执行启动指令

    ```java
    java -jar springboot.jar
    ```
    
3. 注意事项

    jar支持命令行启动需要依赖maven插件支持，请确认打包时是否具有SpringBoot对应的maven插件

    ```xml
    <build>
    	<plugins>
        	<plugin>
            	<groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```

- 使用Jetty启动

  ```xml
  <dependencies>
  	<dependency>
      	<groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
          <!--web起步依赖环境中，排除Tomcat起步依赖-->
          <exclusions>
          	<exclusion>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-tomcat</artifactId>
              </exclusion>
          </exclusions>
      </dependency>
      
      <dependency>
      	<groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-jetty</artifactId>
      </dependency>
  </dependencies>
  ```

  Jetty比Tomcat更轻量级，可扩展性更强（相较于Tomcat），谷歌应用引擎（GAE）已经全面切换为Jetty

# 三、REST风格

REST(Representation State Transfer)，表现形式状态转换

- 传统风格资源描述形式

  http://localhost/user/getById?id=1

  http://localhost/user/saveUser

- REST风格描述形式

  http://localhost/user/1

  http://localhost/user

- 优点

  - 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
  - 书写简化

- 按照REST风格访问资源时使用行为动作区分对资源进行了何种操作

|           URL            |       含义       | 行为动作 |
| :----------------------: | :--------------: | :------: |
|  http://localhost/users  | 查询全部用户信息 |   GET    |
| http://localhost/users/1 | 查询指定用户信息 |   GET    |
|  http://localhost/users  |   添加用户信息   |   POST   |
|  http://localhost/users  |   修改用户信息   |   PUT    |
| http://localhost/users/1 |   删除用户信息   |  DELETE  |

- 根据REST风格进行资源访问成为RESTful

```java
// 新增
@RequestMapping(value = "/users", method = RequestMethod.POST)
public String save();

// 删除指定
@RequestMapping(value = "/users/{id}", method = RequestMethod.DELETE)
public String delete(@PathVariable Interger id);

// 修改
@RequestMapping(value = "/users", method = RequestMethod.PUT)
public String update(@PathVariable User user);

// 查询指定
@RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
public String getById(@PathVariable Interger id);

//查询全部
@RequestMapping(value = "/users", method = RequestMethod.GET)
public String getAll();
```

- 设定http请求动作
- 设定请求变量（路径参数）

# 四、基础配置

### 4.1配置文件格式

- 修改服务器端口

  ```http
  http://localhost:8080/books/1
  ```

  Spring提供了多种属性配置方式

  - application.properties（优先级高）

    ``server.port=80``

  - application.yml（优先级中）

    ```yml
    server:
      port: 81
    ```

  - application.yaml（优先级低）

    ```yaml
    server:
      port: 82
      
    logging:
      level:
        root: info
    ```

  自动提示功能消失解决方案

  File->Project Structure->Facet->Spring->Customize SpringBoot->+

### 4.2 yaml

- YAML (YAML Ain't Markup Language)，一种数据序列化格式

- 优点

  - 容易阅读
  - 容易与脚本语言交互
  - 以数据为核心，重数据清格式

- YAML文件扩展名

  - .yml（主流）
  - .yaml

- 各数据文件格式区别

  - xml

    ```xml
    <enterprise>
    	<name>example</name>
        <age>16</age>
        <tel>4008123123</tel>
    </enterprise>
    ```

  - properties

    ```properties
    enterprise.name=example
    enterprise.age=16
    enterprise.tel=4008123123
    ```

  - yaml

    ```yaml
    enterprise:
    	name: example
    	age: 16
    	tel: 4008123123
    ```

- yaml语法规则

  - 大小写敏感
  - 属性层级关系使用多行描述，每行结尾使用冒号结束
  - 使用缩进表示层级递进关系，同层级左侧对齐，只允许使用空格（不允许使用TAB键）
  - 属性值前添加空格（属性名与属性值之间使用冒号+空格作为分隔符）
  - #表示注释

- yaml数组数据

  ​	数组数据在数据书写位置的下方使用减号作为数据开始符号，每行书写一个数据，减号与数据间空格分割

  ```yml
  enterprise:
  	name: example
  	age: 16
  	tel: 4008123123
  	subject:
  		- Java
  		- 前端
  		- 大数据
  ```

- yaml数据读取

  - 使用@Value读取单个数据，属性名引用方式：``${一级属性名.二级属性名}``
  - 封装全部数据到Envoirnment对象
  - 自定义对象封装指定数据

  ```java
  @Component
  @ConfigurationProperties(prefix = "enterprise")
  public class Enterprise {
      private String name;
      private Integer age;
      private String tel;
      private String[] subject;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public Integer getAge() {
          return age;
      }
  
      public void setAge(Integer age) {
          this.age = age;
      }
  
      public String getTel() {
          return tel;
      }
  
      public void setTel(String tel) {
          this.tel = tel;
      }
  
      public String[] getSubject() {
          return subject;
      }
  
      public void setSubject(String[] subject) {
          this.subject = subject;
      }
  
      @Override
      public String toString() {
          return "Enterprise{" +
                  "name='" + name + '\'' +
                  ", age=" + age +
                  ", tel='" + tel + '\'' +
                  ", subject=" + Arrays.toString(subject) +
                  '}';
      }
  }
  ```

  ```java
  @RestController
  @RequestMapping("/books")
  public class BookController {
      @Value("${lesson}")
      private String lesson;
  
      @Value("${enterprise.subject[0]}")
      private String subject;
  
      @Autowired
      private Environment environment;
  
      @Autowired
      private Enterprise enterprise;
  
      @RequestMapping("/{id}")
      public String getById(@PathVariable Integer id){
          System.out.println("id==>" + id);
          System.out.println(lesson);
          System.out.println(subject);
          System.out.println("----------------------");
          System.out.println(environment.getProperty("lesson"));
          System.out.println(environment.getProperty("server.port"));
          System.out.println(environment.getProperty("enterprise.subject[1]"));
          System.out.println("=======================");
          System.out.println(enterprise);
          return Integer.toString(id);
      }
  }
  ```

- 自定义对象封装数据警告解决方案

  ```java
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <optional>true</optional>
  </dependency>
  ```

### 4.3 多环境启动

- yaml过时格式

  ```yaml
  spring:
    profiles:
      active: dev
  
  lesson: springboot
  enterprise:
    name: example
    age: 13
    tel: 4008123123
    subject:
      - Java
      - 大数据
      - 前端
  
  ---
  server:
    port: 80
  spring:
    profiles: dev
  
  ---
  server:
    port: 81
  spring:
    profiles: pro
  
  ---
  server:
    port: 82
  spring:
    profiles: test
  ```

- yaml推荐格式

  ```yaml
  spring:
    profiles:
      active: dev
  
  lesson: springboot
  enterprise:
    name: example
    age: 13
    tel: 4008123123
    subject:
      - Java
      - 大数据
      - 前端
  
  ---
  server:
    port: 80
  spring:
    config:
      activate:
        profiles: dev
  ```

- properties文件多环境启动

  - 主启动配置文件application.properties

    ``spring.profiles.activate=pro``

  - 环境分类配置文件application-__pro__.properties

    ``server.port=80``

  - 环境分类配置文件application-__dev__.properties

    ``server.port=81``

  - 环境分类配置文件application-__test__.properties

    ``server.port=82``
  
- 多环境启动命令

  - 带参数启动SpringBoot

    ``java -jar springboot.jar --spring.profiles.active=test``

    ``java -jar springboot.jar --server.port=88``

    ``java -jar springboot.jar --server.port=88 --spring.profiles.active=test``

- 参数加载优先顺序

  - 参见[官网](https://docs.spring.io/spring-boot/docs/2.4.13/reference/html/spring-boot-features.html#boot-features-external-config)

- 多环境开发控制（maven、SpringBoot）

  - maven中设置多环境属性

    ```xml
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <profile.active>dev</profile.active>
            </properties>
        </profile>
    
        <profile>
            <id>pro</id>
            <properties>
                <profile.active>pro</profile.active>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
    
        <profile>
            <id>test</id>
            <properties>
                <profile.active>test</profile.active>
            </properties>
        </profile>
    </profiles>
    ```

  - SpringBoot中引用maven属性

    ```yaml
    spring:
      profiles:
        active: ${profile.active}
    
    ---
    server:
      port: 80
    spring:
      profiles: dev
    
    ---
    server:
      port: 81
    spring:
      profiles: pro
    
    ---
    server:
      port: 82
    spring:
      profiles: test
    ```

  - Maven指令执行完毕后，生成了对应的包，其中类参与编译，但是配置文件没有参与编译，而是复制到包中

  - 解决思路：对于源码中非java类的操作要求加载maven对应的属性，解析${}占位符

    ```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <useDefaultDelimiters>true</useDefaultDelimiters>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```

### 4.4 配置文件分类

- SpringBoot中4级配置文件
  - 1级：file: config/application.yml （最高）
  - 2级：file: application.yml
  - 3级：classpath: config/application.yml
  - 4级：classpath: application.yml （最低）

- 作用
  - 1级与2级留作系统打包后设置通用属性
  - 3级与4级用于系统开发阶段设置通用属性

# 五、SpringBoot整合第三方技术

### 5.1 整合JUnit

- Spring整合JUnit

  ```java
  @RunWith(SpringJUnitClassRunner.class)
  @ContextConfiguration(class = SpringConfig.class)
  public class UserServiceTest{
      @AutoWired
      private BookService bookService;
      
      @Test
      public void testSave(){
          bookService.save();
      }
  }
  ```

- SpringBoot整合JUnit

  ```java
  @SpringBootTest
  class SpringbootDemoApplicationTests {
  
      @Autowired
      private BookService bookService;
  
      @Test
      public void testSave() {
          bookService.save();
      }
  
  }
  ```

  - 名称：@SpringBootTest

  - 类型：测试类注解

  - 位置：测试类定义上方

  - 作用：设置JUnit加载的SpringBoot启动类

  - 范例

    ```java
    @SpringBootTest(classes = SpringbootDemoApplication.class)
    class SpringbootDemoApplicationTests {}
    ```

  - 相关属性

    - classes：设置SpringBoot的启动类

  - 注意事项

    - 如果测试类在SpringBoot启动类的包或子包中，可以省略启动类的设置，也就是省略classes的设定
    - 若报错NullPointerException，则在测试类的上方添加注解``@Runwith(SpringRunner.class)``

### 5.2 基于SpringBoot实现SSM整合

- Spring整合MyBatis

  - SpringConfig

    - 导入JdbcConfig
    - 导入MyBatisConfig

    ```java
    @Configuration
    @ComponentScan("com.example")
    @PropertySource("classpath:jdbc.properties")
    @Import(JdbcConfig.class, MyBatisConfig.class)
    public class SpringConfig{
        
    }
    ```

  - JDBCConfig

    - 定义数据源（加载properties配置项：driver, url, username, password）

    ```java
    public class JDBCConfig{
        @Value("${jdbc.driver}")
        private String driver;
        @Value("${jdbc.url}")
        private String url;
        @Value("${jdbc.username}")
        private String userName;
        @Value("${jdbc.password}")
        private String password;
        
        @Bean
        public DataSource getDataSource(){
            DruidDataSource ds = new DruidDataSource();
            ds.setDriverClassName(driver);
            ds.setUrl(url);
            ds.setUserName(username);
            ds.setPassword(password);
            return ds;
        }
    }
    ```

  - MyBatisConfig

    - 定义sqlSessionFactoryBean
    - 定义映射配置

    ```java
    @Bean
    public SqlSessionFactoryBean getSqlSessionFactoryBean(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasPackage("com.exapmle.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }
    ```

    ```java
    @Bean
    public MapperScannerConfiguer getMapperScannerCondfiger(){
        MapperScannerConfiger msc = new MapperScannerConfiger();
        msc.setBasePackage("com.example.dao");
        return msc;
    }
    ```

- SpringBoot整合MyBatis

  1. 创建新模块，选择Spring初始化，并配置模块相关基础信息

     new module -> spring initializer -> java version 8 -> 勾选SQL中的MyBatis framework和MySQL Driver 

  2. 在yaml文件中设置数据源参数

     ```yaml
     spring:
       datasource:
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql://localhost:3306/ssm_db
         username: root
         password: 123456
         type: com.alibaba.druid.pool.DruidDataSource
     
     ```

     注意事项：

     Spring版本低于2.4.3（不含），MySQL驱动版本大于8.0时，需要在url连接串中配置时区``jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC``，或者在MySQL数据库端配置时区解决此问题

  3. 定义数据层接口与映射配置

     ```java
     @Mapper
     public interface BookDao {
         @Select("select * from tbl_book")
         public Book getById(Integer id);
     }
     ```

  4. 测试类中注入dao接口，测试功能

     ```java
     @SpringBootTest
     class SsmDemoApplicationTests {
         @Autowired
         private BookDao bookDao;
     
         @Test
         void contextLoads() {
             Book book = bookDao.getById(1);
             System.out.println(book);
         }
     
     }
     ```

### 5.3 基于SpringBoot的SSM整合案例

1. pom.xml

   配置起步依赖（Spring Web, MyBatis Framework, MySQL Driver），必要的资源坐标（driud）

2. application.yml

   设置数据源、端口等

3. 配置类

   全部删除

4. dao

   设置@Mapper

5. 测试类

   测试类注解改为@SpringBootTest，@Test注解需要换引用包

6. 页面

   将页面的四个文件夹（css, js, pages, plugins）放入resources->static文件夹下

