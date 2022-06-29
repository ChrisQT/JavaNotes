# 一、JavaWeb

### 1. 数据库（略）

### 2. JDBC

JDBC是用JAVA操作关系型数据库的一套API

一组标准接口，可以操作所有支持JDBC的关系型数据库

各数据库厂商使用相同的接口，JAVA代码不需要针对不同的数据库分别开发，可随时更换底层数据库，无需改变JAVA代码。

#### 2.1 快速入门

```java
//注册驱动（通过静态代码块，MySQL5之后可不写）
Class.forName("com.mysql.jdbc.Driver");
//获取链接
Connection conn = DriverManager.getConnnection(url, username, password);
//定义SQL语句
String sql = "update ...";
//获取执行SQL对象
Statement stmt = conn.createStatement();
//执行SQL
stmt.excecuteUpdate(sql);
//处理返回结果
//释放资源
```

#### 2.2 API详解

DriverManager类

1. 注册驱动

2. 获取数据库连接

Connection

​	1.获取执行SQL对象

​	2.管理事务

```java
//开启事务
setAutoCommit(boolean autoCommit);//ture为自动提交事务，false为手动提交，即开启事务
//提交事务
commit();
//回滚事务
rollback();
```

Statement

- 执行SQL语句

```java
int executeUpdate(sql): //执行DML、DDL语句
//返回值：DML为影响的行数，DDL执行成功可能为0
ResultSet excecuteQuery(sql): //执行DQL语句
//返回值：ResultSet对象
```

ResultSet

1. 封装DQL的查询结果

```java
ResultSet stmt.excecuteQuery(sql): // 执行DQL语句，返回ResultSet对象
```

2. 获取查询结果

```java
boolean next(): //将光标往下移一行，需判断当前行是否为有效行
xxx getXxx(参数): //获取数据，xxx为数据类型。参数：列的编号（1开始）/列的名称（String） 
```

PreparedStatement

作用：预编译SQL语句并执行SQL语句

1. 获取PreparedStatement对象

```java
//使用?作为占位符替代用户输入内容
String sql = "select * from user where username = ? and password = ?";
//通过Connection对象获取，并传入对应的SQL语句
PreparedStatement pstmt = conn.preparedStatement(sql);
```

2. 设置参数值

```java
PreparedStatement对象：setXxx(参数1, 参数2); //给?赋值，Xxx为数据类型
// 参数1：参数的编号（1开始），参数2：参数的值
```

3. 执行SQL

```java
excecuteUpdate(); 
excecuteQuery();
```

*_开启预编译功能需要在url中添加：useServerPreStmts=true_

### 3.数据库连接池

容器，负责分配，管理数据库连接。允许程序复用现有的数据库连接，而非反复创建和删除。

好处：资源复用、提升系统响应速度、避免数据库连接遗漏

####  3.1 标准接口DataSource

功能：获取连接

官方提供标准接口，__由第三方组织实现__

#### 3.2 Druid

阿里开源，常用连接池

1. 导入jar包
2. 定义配置文件
3. 加载配置文件
4. 获取数据库连接池对象
5. 获取连接

```java
System.out.println(System.getProperty("user.dir"));
Properties prop = new Properties();
prop.load(new FileInputStream("JDBC-demo/src/druid.properties"));
DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);
Connection conn = dataSource.getConnection();
System.out.println(conn);
```

alt+insert快速生成toString()方法

### 4. Maven

专门用于管理和构建Java项目的工具，主要功能有：

- 提供了一套标准项目结构（使不同IDE创建的Java项目结构一致且通用）
- 提供了一套标准构建流程（编译工具等组件）
- 提供了一套依赖管理机制（本地仓库、中央仓库、远程仓库，类似python的官方库/镜像源）

#### 4.1 常用命令

- mvn compile
- mvn clean
- mvn package
- mvn test
- mvn install

#### 4.2 生命周期

- clean: 清理工作
- default: 核心工作，如编译、测试、打包、安装等等
- site: 产生报告，发布站点（不常用）



- 同一生命周期内，执行后面的命令，前面的命令会自动执行
  - pre-clean > clean > post-clean
  - compile > test > package > install
  - pre-site > site > post-site 

#### 4.3 IDEA配置Maven

1. 创建模块，选择Maven，点击Next
2. 填写模块名称，坐标信息，点击finish

#### 4.4 IDEA导入Maven

1. 选择右侧Maven面板，点击+号（若无则View -> Appearence -> Tool Window Bars）
2. 选中需导入项目的pom.xml文件，双击

#### 4.5 依赖管理

- 在pom.xml使用<dependencies>和<dependency>引入坐标

- alt+insert 快速导入
- 依赖范围：通过<scope>管理包生效的范围：compile, test, provided, runtime, system, import

|  依赖范围   | 编译 | 测试 | 运行 |
| :---------: | :--: | :--: | :--: |
| __compile__ |  Y   |  Y   |  Y   |
|    test     |  -   |  Y   |  -   |
|  provided   |  Y   |  Y   |  -   |
|   runtime   |  -   |  Y   |  Y   |
|   system    |  Y   |  Y   |  -   |

### 5. MyBatis

JavaEE三层架构：表现层、业务层、持久层。持久层负责将数据保存到数据库。

MyBatis是一个持久层框架，简化JDBC开发。

- JDBC缺点：
  - 硬编码：用户名密码SQL语句等配置信息写死在程序代码中 -> 配置文件
  - 封装结果集过于繁琐 -> 自动完成
- JDBC原版：

```java
//1. 注册驱动
Class.forName("com.mysql.jdbc.Driver");
//2. 获取Connection连接
String url = "jdbc:mysql:///db?useSSL=false";
String user = "root";
String password = "123456"
Connection conn = DriverManager.getConnection(url, user, password);
//定义SQL
String sql = "select * from table where age = ?";
//获取pstmt对象
PreparedStatement pstmt = conn.prepareStatement(sql);
//设置?的值
pstmt.setInt(1, 10086);
//执行SQL
ResultSet rs= pstmt.executeQuery();
//遍历结果集
User user = null;
ArrayList<User> users = new ArrayList<>();
while(rs.next()){
    //获取数据
    int id  = rs.getInt("id");
    String username = rs.getString("username");
    String password = rs.getString("passsword");
    
    //创建对象封装结果（ORM,对象关系映射）
    user = new User();
    user.setId(id);
    user.setUsername(username);
    user.setPassword(password);
    
    //插入集合
    users.add(user);
}
res.close();
pstmt.close();
conn.close();
```

- Mybatis步骤
  - 创建Maven模块，导入坐标
  - 编写Mybatis核心配置文件 ->替换连接信息，解决硬编码问题
  - 编写SQL映射文件->统一管理SQL语句，解决硬编码问题
  - 编码：
    - 定义POJO类（简单JAVA对象）
    - 加载核心配置文件，获取SqlSessionFactory对象
    - 获取SqlSession对象，执行SQL语句
    - 释放资源

```java
// 1.加载mybatis的核心配置文件，获取sqlSessionFactory
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

// 2.获取sqlSession对象，用它执行SQL语句
SqlSession sqlSession = sqlSessionFactory.openSession();

// 3.执行SQL语句
List<Employee> emps = sqlSession.selectList("test.selectFirstTen");
System.out.println(emps);

// 4.释放资源
sqlSession.close();
```

#### 5.1 Mapper代理开发

解决原生方式中的硬编码，简化后期执行SQL。原先：

```java
List<Employee> emps = sqlSession.selectList("test.selectFirstTen");
System.out.println(emps);
```

使用Mapper开发：

```java
EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class)
List<Employee> emps = employeeMapper.selectFirstTen();
```

- 使用Mapper代理完成入门案例：
  1. 定义与SQL映射文件同名的Mapper接口，并且将Mapper接口和SQL映射文件放置于同一目录下
  2. 设置SQL映射文件的namespace属性为Mapper的接口全限定名
  3. 在Mapper接口中定义方法，方法名为SQL映射文件中sql语句的id，并保持参数类型和返回值类型一致。
  4. 编码
     1. 通过SqlSession的getMapper方法获取Mapper接口的代理对象。
     2. 调用对应方法完成sql的执行
- TIPS：若Mapper接口名称和SQL映射文件名称相同，并在同一目录下，则可以使用包扫描的的方式简化SQL映射文件的加载

```html
<mappers>
    <mapper resource="com/xjtu/mapper/EmployeeMapper.xml"/>
    <mapper resource="com/xjtu/mapper/XXX.xml"/>
    <mapper resource="com/xjtu/mapper/YYY.xml"/>
</mappers>
```

或者：

```html
<mappers>
    <package name="com.xjtu.mapper"/>
</mappers>
```

#### 5.2 MyBatis配置

- [MyBatis核心配置文件详解](https://mybatis.org/mybatis-3/zh/configuration.html)

- 配置文件完成增删改查
  1. 编写接口方法：Mapper接口   ```List<Employee> selectAll();```
  2. 编写SQL语句：SQL映射文件  ``<select id="selectAll" resultType="employee"> select * from employees </select> ``
  3. 执行方法，测试

*__数据库表的字段名称和POJO的属性名称不一样时，MyBatis的自动封装会出问题！__解决方法：

- SQL起别名 ``SELECT emp_id as empId``(不推荐)
- 在pom文件中用ResultMap完成列名和属性之间的映射

```java
<mapper namespace="com.xjtu.mapper.EmployeeMapper">
    <resultMap id="employeeResultMapper" type="com.xjtu.pojo.Employee">
        <result column="emp_no" property="empNo"/>
        <result column="birth_date" property="birthDate"/>
        <result column="first_name" property="firstName"/>
        <result column="last_name" property="lastName"/>
        <result column="hire_date" property="hireDate"/>

    </resultMap>

    <select id="selectFirstTen" resultMap="employeeResultMapper">
        select * from employees limit 10;
    </select>
</mapper>
```

*MyBatis中写SQL的参数占位符：

- #{}：会将其替换为？，可以防止SQL注入
- ${}：拼SQL，会存在SQL注入问题，表名或列名不固定时可用

*特殊字符的处理：

​	1. 转义字符 ：'<'可用'$lt'代替

	2. CDATA区<![CDATA[特殊字符]]>

#### 5.3 使用Mybatis完成增删改查

- SQL多条件查询

```java
// 标明参数
List<Employee> selectByCondition(@Param("status")int status, @Param("FirstName")String firstName, @param("gender")String gender);
// 参数名和xml中一致
List<Employee> selectByCondition(Employee employee);

List<Employee> selectByCondition(Map map); 
```

```html
<select id="selectByCondition" resultMap="employeeMap">
    select * from employees where status = #{status}
    and first_name Like #{firstname}
    and gender = #{gender}
</select>
```

- MyBatis的动态SQL支持
  - 多条件动态查询

```html
<select id="selectByCondition" resultMap="employeeResultMapper">
    select * from employees where 1=1
    <if test="gender != null and gender != ''">
        and gender = #{gender}
    </if>
    <if test="firstName != null and firstName != ''">
        and first_name like #{firstName}
    </if>
</select>
```

或者

```html
<select id="selectByCondition" resultMap="employeeResultMapper">
    select * from employees
    <where>
        <if test="gender != null and gender != ''">
            and gender = #{gender}
        </if>
        <if test="firstName != null and firstName != ''">
            and first_name like #{firstName}
        </if>
    </where>
</select>
```

- 单条件动态查询

  choose(when,otherwise)，逻辑类似Java的switch语句

```html
<select id="selectByCondition" resultMap="employeeResultMapper">
    select * from employees where
    <choose>
        <when test="firstName != null and firstName != ''">
            first_name like #{firstName}
        </when>
        <when test="gender != null and gender != ''">
        	gender=#{gender}
        </when>
        <otherwise>
            1 = 1
        </otherwise>
    </choose>
</select>
```

或者

```html
<select id="selectByCondition" resultMap="employeeResultMapper">
    select * from employees
    <where>
        <choose>
            <when test="firstName != null and firstName != ''">
                first_name like #{firstName}
            </when>
            <when test="gender != null and gender != ''">
                gender=#{gender}
            </when>
        </choose>
    </where>
</select>
```

- MyBatis默认开启事务，因此修改操作需要手动提交。

```java
//开启连接
SqlSession sqlSession = sqlSessionFactory.openSession();
EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
//默认开启事务
employeeMapper.add(emp);
//手动提交
sqlSession.commit();
//释放资源
sqlSession.close();
```

或者设置自动提交事务

```java
SqlSession sqlSession = sqlSessionFactory.openSession(autocommit:true);
```

- 当未提交时如何获得新增记录的主键：

```html
<insert useGeneratedKeys="true" keyProperty="id">SQL语句</insert>
```

```java
employeeMapper.add(emp);
Integer empNo = emp.getEmpNo();
```

- MyBatis修改操作的动态SQL支持

```html
<update id="set">
    update employees
    <set>
        <if test="birthDate != null and birthDate != ''">
            birth_date = #{birthDate},
            </if>
        <if test="firstName != null and firstName != ''">
            first_name = #{firstName},
        </if>
        <if test="lastName != null and lastName != ''">
            last_name = #{lastName},
        </if>
        <if test="gender != null and gender != ''">
            gender = #{gender},
        </if>
        <if test="hireDate != null and hireDate != ''">
            hire_date = #{hireDate}
        </if>
        where emp_no = #{empNo}
    </set>
</update>
```

- MyBatis删除操作的动态SQL支持

```java
employeeMapper.deleteByIds(ids);
```

```html
<!--
	MyBatis会将数组参数，封装为一个Map集合。
		* 默认：array = 数组
		* 使用@Params注解改变map集合的默认key的名称
-->
<delete id="deleteByIds">
    delete from employees
    where emp_no in
    <foreach collection="array" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</delete>
```

#### 5.4 MyBatis参数传递

MyBatis接口方法可以接受各种各样的参数，MyBatis底层对于这些参数进行不同的封装处理方式

- 单个参数：

  - POJO类型：直接使用，属性名和参数占位符一致

  - Map集合：直接使用，属性名和参数占位符一致

  - Collection：封装成Map集合，可以用@Param注解

    ``map.put("arg0", collection集合);``

    ``map.put("collection", collection集合);``

  - List：封装成Map集合，可以用@Param注解

    ``map.put("arg0", list集合);``

    ``map.put("collection", list集合);``

    ``map.put("list", list集合);``

  - Array：封装成Map集合，可以用@Param注解

    ``map.put("arg0", array数组);``

    ``map.put("array", array数组);``

  - 其他类型：直接使用

- 多个参数

  MyBatis会将参数封装为Map集合，可以使用@Param注解，替换Map集合中的默认键名。

```java
map.put("arg0", 参数值1);
map.put("param1", 参数值1);
map.put("arg1", 参数值2);
map.put("param2", 参数值2);
```

```java
User select(@Param("username")String username, @Param("password")String Password)
```

#### 5.5 注解完成增删改查

使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。

```java
@Select("select * from employees where emp_no = #{empNo}")
```