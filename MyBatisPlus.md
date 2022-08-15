# 一、MyBatisPlus简介

- MyBatisPlus（简称MP）是基于MyBatis框架基础上开发的增强型工具，旨在简化开发、提高效率
- 开发方式
  - 基于MyBatis使用MyBatisPlus
  - 基于Spring使用MyBatisPlus
  - __基于SpringBoot使用MyBatisPlus__

### 1.1 SpringBoot整合MyBatis开发过程

- 创建SpringBoot工程
- 勾选配置使用的技术
- 设置dataSource相关属性（JDBC参数）
- 定义数据层接口映射配置

### 1.2 SpringBoot整合MyBatisPlus入门程序

1. 创建新模块，选择Spring初始化，并配置模块相关基础信息

2.  选择当前模块需要使用的技术集（仅保留JDBC）

3. 手动添加MyBatisPlus起步依赖

   ```xml
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.4.1</version>
   </dependency>
   ```

   由于MyBatisPlus并未收录到idea的系统内置配置，无法直接选择加入

4. 设置Jdbc参数（application.yml）

   ```yaml
   spring:
     datasource:
       username: root
       password: 123456
       url: jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
   ```

   如果使用druid数据源，需要导入对应坐标

5. 制作实体类与表结构（类名与表名对应，属性名与字段名对应）

6. 定义数据接口，继承``BaseMapper<User>``

   ```java
   @Mapper
   public interface UserDao extends BaseMapper<User> {
   }
   ```

7. 测试类中注入dao接口，测试功能

   ```java
   @SpringBootTest
   class MybatisDemoApplicationTests {
       @Autowired
       private UserDao userDao;
   
       @Test
       void testGetAll(){
           List<User> userList = userDao.selectList(null);
           System.out.println(userList);
       }
   }
   ```

### 1.3 MyBatisPlus概述

- MyBatisPlus（简写MP）是基于MyBatis框架基础上开发的增强型工具，旨在简化开发、提高效率
- 官网：https://mybatis.plus/  http://mp.baomidou.com/

- 特性
  - 无侵入：只作增强不做改变，不会对现有的工程产生影响
  - 强大的CRUD操作：内置通用Mapper，少量配置即可实现单表CRUD操作
  - 支持lambda：编写查询系统无需担心字段写错
  - 支持主键自动生成
  - 内置分页插件

# 二、标准数据层开发

### 2.1 标准数据层CRUD功能

|    功能    |                 自定义接口                 | MP接口                                           |
| :--------: | :----------------------------------------: | ------------------------------------------------ |
|    新增    |           ``boolean save(T t)``            | ``int insert(T t)``                              |
|    删除    |         ``boolean delete(int id)``         | ``int deleteById(Serializable id)``              |
|    修改    |          ``boolean update (T t)``          | ``int updateById(T t)``                          |
| 根据id查询 |           ``T getById(int id)``            | ``T selectById(Serializable id)``                |
|  查询全部  |           ``List<T> getAll() ``            | ``List<T> selectList()``                         |
|  分页查询  | ``PageInfo<T> getAll(int page, int size)`` | ``IPage<T> selectPage(IPage<T> page)``           |
| 按条件查询 |  ``List<T> getAll(Condition condition)``   | ``IPage<T> selectPage(Wrapper<T> queryWrapper)`` |

- Lombok

  一个Java类，提供了一组注解，简化POJO实体类开发

  ```xml
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
  </dependency
  ```

  ```java
  @Setter
  @Getter
  @ToString
  @AllArgsConstructor
  @NoArgsConstructor
  @EqualsAndHashCode
  public class User {
      private long id;
      private String name;
      private String password;
      private Integer age;
      private String tel;
  }
  ```

  或者使用常用注解``@Data``，为当前实体类在编译期设置对应的get/set方法，有参/无参构造方法，toString方法，hashCode方法，equals方法等

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {
      private long id;
      private String name;
      private String password;
      private Integer age;
      private String tel;
  }
  ```

### 2.2 分页功能

1. 设置分页拦截器作为Spring管理的bean

   ```java
   @Configuration
   public class MpConfig {
       @Bean
       public MybatisPlusInterceptor pageInterceptor(){
           MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
           interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
           return interceptor;
       }
   }
   ```

2. 执行分页查询

   ```java
   @SpringBootTest
   class MybatisDemoApplicationTests {
       @Autowired
       private UserDao userDao;
   
       @Test
       void selectGetPage(){
           IPage page = new Page(1,2);
           userDao.selectPage(page, null);
           System.out.println("Current page No.: " + page.getCurrent());
           System.out.println("Records per page: " + page.getSize());
           System.out.println("Num of Pages: " + page.getPages());
           System.out.println("Num of records: " + page.getTotal());
           System.out.println("Records: " + page.getRecords());
       }
   }
   ```

3. （可选）开启日志方便调试

   ```yaml
   # 开启MP日志到控制台
   mybatis-plus:
     configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
   ```


# 三、DQL编程控制

### 3.1 条件查询方式

- MyBatisPlus将书写复杂的SQL查询条件进行了封装，使用编程的形式完成查询的组合

- 方式一：

  ```java
  @Test
  void CondQuery(){
      QueryWrapper qw = new QueryWrapper();
      qw.lt("age", 25);
      qw.gt("age", 15);
      List<User> userList = userDao.selectList(qw);
      System.out.println(userList);
  }
  ```

  或者链式编程

  ```java
  @Test
  void CondQuery(){
      QueryWrapper qw = new QueryWrapper();
      qw.lt("age", 25).gt("age", 15);
      List<User> userList = userDao.selectList(qw);
      System.out.println(userList);
  }
  ```

- 方式二

  ```java
  @Test
  void CondQuery(){
      QueryWrapper<User> qw = new QueryWrapper();
      qw.lambda().lt(User::getAge, 18).gt(User::getAge, 13);
      List<User> userList = userDao.selectList(qw);
      System.out.println(userList);
  }
  ```

- __方式三（推荐）__

  ```java
  @Test
  void CondQuery(){
      LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
      lqw.lt(User::getAge, 28).gt(User::getAge, 26);
      List<User> userList = userDao.selectList(lqw);
      System.out.println(userList);
  }
  ```

  表示“或”使用or()

  ```java
  @Test
  void CondQuery(){
      LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
      lqw.gt(User::getAge, 28).or().lt(User::getAge, 26);
      List<User> userList = userDao.selectList(lqw);
      System.out.println(userList);
  }
  ```

- 条件查询null值处理

  - if语句控制条件追加

    ```java
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
    if(null != userQuery.getAge()){
        lqw.gt(User::getAge, userQuery.getAge());
    }
    if(null != userQuery.getAge2()){
        lqw.lt(User::getAge, userQuery.getAge2());
    }
    List<User> userList = userDao.selectList(lqw);
    System.out.println(userList);
    ```

  - __条件参数控制（推荐）__

    ```java
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
    lqw.gt(null!=uq.getAge(), User::getAge, uq.getAge());
    lqw.lt(null!=uq.getAge2(), User::getAge, uq.getAge2());
    List<User> userList = userDao.selectList(lqw);
    System.out.println(userList);
    ```

  - 条件参数控制（链式编程）

    ```java
    LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
    lqw.gt(null!=uq.getAge(), User::getAge, uq.getAge())
       .lt(null!=uq.getAge2(), User::getAge, uq.getAge2());
    List<User> userList = userDao.selectList(lqw);
    System.out.println(userList);
    ```


### 3.2 查询投影

- 查询结果包含模型部分属性

  ```java
  @Test
  void CondQuery3(){
      LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
      lqw.select(User::getAge, User::getId, User::getName);
      List<User> userList = userDao.selectList(lqw);
      System.out.println(userList);
  }
  ```

  或者

  ```java
  @Test
  void CondQuery4(){
      QueryWrapper<User> lqw = new QueryWrapper();
      lqw.select("age", "id", "name");
      List<User> userList = userDao.selectList(lqw);
      System.out.println(userList);
  }
  ```

- 查询结果包含模型类中未定义的属性

  ```java
  @Test
  void CondQuery5(){
      QueryWrapper<User> lqw = new QueryWrapper();
      lqw.select("count(*) as count, tel");
      lqw.groupBy("tel");
      List<Map<String, Object>> userList = userDao.selectMaps(lqw);
      System.out.println(userList);
  }
  ```

- 查询条件设定

  - 范围匹配（>、<、=、between）

  - 模糊匹配（like）

    ```java
    @Test
    void CondQuery8(){
        LambdaQueryWrapper<User> lqw = new LambdaQueryWrapper();
        lqw.like(User::getName, "J");
        List<User> userList = userDao.selectList(lqw);
        System.out.println(userList);
    }
    ```

  - 空判定（null）

  - 包含性匹配（in）

  - 分组（group）

    ```java
    @Test
    void GroupQuery(){
        QueryWrapper<User> qw = new QueryWrapper();
        qw.select("gender", "count(*) as nums");
        qw.groupBy("gender");
        List<Map<String, Object>> maps = userDao.selectMaps(qw);
        System.out.println(maps);
    }
    ```

  - 排序（order）

  - 更多查询条件设置请查看http://mp.baomidou.com/

### 3.3 字段映射与表名映射

- 问题一：表字段名与编码属性不同步

  ```java
  public class User{
      private Long id;
      private String name;
      @TableField(value="pwd")
      private String password;
      private Integer age;
      private String tel;    
  }
  ```

- 问题二：编码中添加了数据库中未定义的属性

  ```java
  public class User{
      private Long id;
      private String name;
      @TableField(value="pwd")
      private String password;
      private Integer age;
      private String tel;
      @TableField(exist=false)
      private Integer online;
  }
  ```

- 问题三：采用默认查询开放了更多的字段查看权限

  ```java
  public class User{
      private Long id;
      private String name;
      @TableField(value="pwd", select=false)
      private String password;
      private Integer age;
      private String tel;
      @TableField(exist=false)
      private Integer online;
  }
  ```

- @TableField注解
    - 名称：``@TableField``
    - 类型:属性注解
    - 位置：模型类属性定义上方
    - 作用：设置当前属性对应的数据库表中的字段关系
    - 相关属性：
      - value（默认）：设置数据库表字段名称
      - exist：设置属性在数据库表字段中是否存在，默认为true。此属性无法与value合并使用
      - select：设置属性是否参与查询，此属性与select()映射配置不冲突

- 问题四：表名与编码开发的类名设计不同步

  ```java
  @TableName("tbl_user")
  public class User{
      private Long id;
      private String name;
      @TableField(value="pwd", select=false)
      private String password;
      private Integer age;
      private String tel;
      @TableField(exist=false)
      private Integer online;
  }
  ```

- @TableName注解

  - 名称：``@TableName``
  - 类型：类注解
  - 位置：模型类定义上方
  - 作用：设置当前类对应与数据库关系
  - 相关属性：value：设置数据库表名称

# 四、DML编程控制

### 4.1 id生成策略控制

```java
public class User{
	@TableId(type = Idtype.AUTO)
	private long id;
}
```

- @TableId注解
  - 名称：@TableId
  - 类型：属性注解
  - 位置：模型类中用于表示主键的属性定义上方
  - 作用：设置当前类中主键属性的生成策略
  - 相关属性
    - value：设置数据库主键名称
    - type：设置主键属性的生成策略，值参照IdType枚举值
      - IdType.AUTO(0)：使用数据库id自增策略控制id生成
      - IdType.NONE(1)：不设置id生成策略
      - IdType.INPUT(2)：用户手动输入Iid
      - IdType.ASSIGN_ID(3)：雪花算法生成id（可兼容数值类型和字符串类型）
      - IdType.UUID(4)：以UUID生成算法作为id生成策略

- 全局配置

  ```yaml
  mybatis-plus:
    configuration:
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    global-config:
      banner: false
      db-config:
        id-type: assign_id
  ```

  

*当表名的前缀相同（tbl_book, tbl_user）时，设置不同的pojo类（Book, User）时，可以通过全局设置的方式统一映射，不用每次都写``@TableName``

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    banner: false
    db-config:
      id-type: assign_id
      table-prefix: tbl_
```

### 4.2 多记录操作

- 按照主键删除多条记录

  ```java
  @Test
  void testDelete(){
      List<Long> list = new ArrayList<>();
      list.add(8L);
      list.add(9L);
      userDao.deleteBatchIds(list);
  }
  ```

- 按照主键查询多条记录

  ```java
  @Test
  void testAddBatch(){
      List<Long> list = new ArrayList<>();
      list.add(1L);
      list.add(2L);
      userDao.selectBatchIds(list);
  }
  ```

### 4.3 逻辑删除

1. 数据库表中添加逻辑删除标记字段

2. 实体类中添加对应字段，并设定当前字段为逻辑删除标记字段

   ```java
   @Data
   @NoArgsConstructor
   @EqualsAndHashCode
   //@TableName("tbl_user")
   public class User {
   //    @TableId(type = IdType.AUTO)
       private long id;
       private String name;
       @TableField(select = false)
       private String password;
       private Integer age;
       private String tel;
       @TableField(exist = false)
       private Integer online;
       @TableLogic(value = "0", delval = "1")
       private Integer deleted;
   }
   ```

   或者配置逻辑删除字段值

   ```yaml
   mybatis-plus:
     configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
     global-config:
       banner: false
       db-config:
         id-type: assign_id
         table-prefix: tbl_
         logic-delete-field: deleted
         logic-delete-value: 1
         logic-not-delete-value: 0
   ```

3. 原先MyBatisPlus的删除接口执行的是逻辑删除字段置0的更新操作，查询时忽略逻辑删除的记录，若想查询上述记录只能手动编写SQL

### 4.4 乐观锁

1. 数据库表中添加锁标记字段

2. 实体类中添加对应字段，并设置当前字段为锁标记字段

   ```java
   public class User{
       private Long id;
       @Version
       private Integer version;
   }
   ```

3. 使用拦截器实现锁机制对应的SQL语句拼装

   ```java
   @Configuration
   public class MpConfig {
       @Bean
       public MybatisPlusInterceptor pageInterceptor(){
           MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
           interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
           interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
           return interceptor;
       }
   }
   ```

4. 使用乐观锁机制在修改前必须先获得对应数据的version方可正常进行

   ```java
   @Test    
   void testOptimisticUpdate(){
       User user = userDao.selectById(3L);
       user.setAge(99);
       userDao.updateById(user);
   }
   ```

   执行修改前先执行查询语句：

   ```sql
   SELECT id,name,age,tel,deleted,version FROM tbl_user WHERE id=? AND deleted=0
   ```

   执行修改时使用version字段作为乐观锁检查依据

   ```sql
   UPDATE tbl_user SET name=?, age=?, tel=?, version=? WHERE id=? AND version=? AND deleted=0
   ```

   

# 五、快速开发

代码生成器

- 模板：MyBatisPlus提供
- 数据库相关配置：读取数据库获取信息
- 开发者自定义配置：手工配置

1. 导入坐标

   ```xml
   <!--Code generator-->
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-generator</artifactId>
       <version>3.4.1</version>
   </dependency>
   
   <!--template engine-->
   <dependency>
       <groupId>org.apache.velocity</groupId>
       <artifactId>velocity-engine-core</artifactId>
       <version>2.3</version>
   </dependency>
   ```

2. 核心代码

   ```java
   public class Generator {
       public static void main(String[] args) {
           AutoGenerator autoGenerator = new AutoGenerator();
   
           // 设置数据源
           DataSourceConfig dataSourceConfig = new DataSourceConfig();
           dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
           dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC");
           dataSourceConfig.setUsername("root");
           dataSourceConfig.setPassword("123456");
           autoGenerator.setDataSource(dataSourceConfig);
   
           // 设置全局配置
           GlobalConfig globalConfig = new GlobalConfig();
           globalConfig.setOutputDir(System.getProperty("user.dir") + "/src/main/java");
           globalConfig.setOpen(false);
           globalConfig.setAuthor("QT");
           globalConfig.setFileOverride(true);
           globalConfig.setMapperName("%sDao");
           globalConfig.setIdType(IdType.ASSIGN_ID);
           autoGenerator.setGlobalConfig(globalConfig);
   
           // 包相关配置
           PackageConfig packageConfig = new PackageConfig();
           packageConfig.setParent("com.example");
           packageConfig.setEntity("domain");
           packageConfig.setMapper("dao");
           autoGenerator.setPackageInfo(packageConfig);
   
           // 策略配置
           StrategyConfig strategyConfig = new StrategyConfig();
           strategyConfig.setInclude("tbl_user");
           strategyConfig.setTablePrefix("tbl_");
           strategyConfig.setRestControllerStyle(true);
           strategyConfig.setEntityLombokModel(true);
           strategyConfig.setLogicDeleteFieldName("delete");
           strategyConfig.setVersionFieldName("version");
           autoGenerator.setStrategy(strategyConfig);
   
           autoGenerator.execute();
       }
   }
   
   ```

   