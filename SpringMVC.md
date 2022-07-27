# 一、SpringMVC

- SpringMVC技术与Servlet技术功能等同，均属于web层开发技术

- 目录
  - SpringMVC简介
  - 请求与响应
  - REST风格
  - SSM整合
  - 拦截器

# 二、 SpringMVC简介

- 浏览器->页面{HTML, CSS, VUE, ElementUI}->后端服务器{表现层SpringMVC、业务层、数据层MyBatis}

- SpringMVC是一种基于Java实现的MVC模型的轻量级Web框架

### 2.1入门案例

1. 导入SpringMVC和Servlet坐标

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```

2. 创建SpringMVC控制类（等同于Servlet功能）

```java
@Controller
public class EmployeeController {
    @RequestMapping("/save")
    @ResponseBody // 设置当前控制器方法相应内容为当前返回值，无需解析
    public String save(){
        System.out.println("employee saved!");
        return "{'info':'springmvc'}";
    }
}
```

3. 初始化SpringMVC环境，设定SpringMVC加载对应的bean

```java
@Configuration
@ComponentScan("com.example.controller")
public class SpringMvcConfig {
}
```

4. 定义一个Servlet容器启动的配置类，在里面加载Spring配置

```java
public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {
    // 加载SpringMVC容器配置
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    // 设置哪些请求归属SpringMVC处理
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    // 加载Spring容器配置
    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }
}
```

5. 运行时添加tomcat插件

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <port>80</port>
                <path>/</path>
<!--            <ignorePackaging>true</ignorePackaging>-->
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 2.2 案例工作流程分析

- 启动服务器初始化过程
  1. 服务器启动，执行ServletContainersInitConfig类，初始化web容器
  2. 执行createServletApplicationContext方法，创建了WebApplicationContext对象
  3. 加载SpringMvcConfig
  4. 执行@ComponentScan加载对应的bean
  5. 加载UserController，每个@RequestMapping的名称对应一个具体的方法
  6. 执行getServletMappings方法，定义所有的请求都通过SpringMVC

- 单次请求执行过程
  1. 发送请求localhost/save
  2. web容器发现所有请求都经过SpringMVC，将请求交给SpringMVC处理
  3. 解析请求路径/save
  4. 由/save匹配执行对应的方法save()
  5. 执行save()
  6. 检测到有@ResponseBody直接将save()方法的返回值作为响应体返回给请求方

- Controller加载控制与业务bean加载控制
  - SpringMVC相关bean（表现层bean）
  - Spring控制的bean
    - 业务bean（Service）
    - 功能bean（DataSource等）
  - 因为功能不同，如何避免Spring错误地加载到SpringMVC的bean的时候排除掉SpringMVC控制的Bean

- SpringMVC相关bean加载控制

  - SpringMVC加载的bean对应的包均在com.example.controller包内

- Spring相关bean加载控制

  - 方式一：Spring加载的bean设定扫描范围为com.example，排除掉controller包内的bean

  ```java
  @Configuration
  @ComponentScan(value="com.example",
      excludeFilters = @ComponentScan.Filter(
          type = FilterType.ANNOTATION,
          classes = Controller.class
      )
  )
  public class SpringConfig{
  }
  ```

  - 方式二：Spring加载的bean设定扫描范围为精准范围，例如service包、dao包等
  - 方式三：不区分Spring与SpringMVC的环境，加载到同一个环境中

- 在Web容器启动时加载Spring容器

```java
public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {
    // 加载SpringMVC容器配置
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    // 设置哪些请求归属SpringMVC处理
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    // 加载Spring容器配置
    @Override
    protected WebApplicationContext createRootApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }
}
```

### 2.3 Web容器配置类的简单写法

```java
public class ServletContainerInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer{

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

### 2.4 PostMan简介

- PostMan是一款网页调试于发送网页HTTP请求的Chrome插件
- 作用：常用于进行接口测试

# 三、请求与响应

### 3.1 请求映射路径

> 名称：@RequestMapping
>
> 类型：方法注解、类注解
>
> 位置：SpringMVC控制器方法定义上方
>
> 作用：设置当前控制器方法请求访问路径，如果设置在类上，统一设置当前控制器方法请求访问路径前缀
>
> 属性：value：请求访问路径或访问路径前缀

- 团队多人开发，每个人设置不同的请求路径，冲突问题如何解决——设置模块名作为请求路径前缀

```java
@Controller
@RequestMapping("/employee")
public class EmployeeController {
    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("employee saved!");
        return "{'info':'springmvc'}";
    }
}
```

### 3.2 请求参数

- Get请求

  普通参数：url地址传参，地址参数名与形参变量相同，定义形参即可接收参数

  ```java
  public class EmployeeController {
      @RequestMapping("/echo")
      @ResponseBody
      public String echo(String name ,int age){
          System.out.println("name = " + name + ", age= " + age);
          return "name = " + name + ", age= " + age;
      }
  }
  ```

- Post请求

  普通参数：form表单post请求传参，表单参数名与形参变量名相同，定义形参即可接收参数

- SpringMVC解决Post请求中的中文乱码问题

  ```java
  public class ServletContainerInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer{
  
      @Override
      protected Class<?>[] getRootConfigClasses() {
          return new Class[]{SpringConfig.class};
      }
  
      @Override
      protected Class<?>[] getServletConfigClasses() {
          return new Class[]{SpringMvcConfig.class};
      }
  
      @Override
      protected String[] getServletMappings() {
          return new String[]{"/"};
      }
  
      // POST乱码处理
      @Override
      protected Filter[] getServletFilters() {
          CharacterEncodingFilter filter = new CharacterEncodingFilter();
          filter.setEncoding("UTF-8");
          return new Filter[]{filter};
      }
  }
  ```

- url参数名和后台Controller方法参数名不一致时

  > 名称：@RequestParam
  >
  > 类型：形参注解
  >
  > 位置：SpringMVC控制器方法形参定义前面
  >
  > 作用：绑定请求参数与处理器方法形参间的关系
  >
  > 参数：required：是否为必传参数；defaultValue：参数默认值

  ```java
  @RequestMapping("/echo")
  @ResponseBody
  public String echo(@RequestParam("name") String userName , int age){
      System.out.println("name = " + userName + ", age= " + age);
      return "name = " + userName + ", age= " + age;
  }
  ```

- 当Controller方法形参为pojo类时，若url属性名和实体类属性名一致，则对应参数可以接收

  ```java
  @RequestMapping("/pojo")
  @ResponseBody
  public String pojoParam(Employee employee){
      System.out.println("Employee: " + employee);
      return "Employee: " + employee;
  }
  ```

  ```http
  http://localhost/employee/pojo?name=chris&age=13
  ```

- 当pojo类中含有引用类型的属性时：

  ```http
  http://localhost/employee/pojo?name=chris&age=13&address.city=beijing&address.province=beijing
  ```

- 当参数中含有数组时：

  ```java
  @RequestMapping("/array")
  @ResponseBody
  public String likes(String[] hobbies){
      System.out.println("hobbies: " + Arrays.toString(hobbies));
      return "hobbies: " + Arrays.toString(hobbies);
  }
  ```

  ```http
  http://localhost/employee/array?hobbies=swim&hobbies=badminton
  ```

- 当参数中含有集合

  ```java
  @RequestMapping("/list")
  @ResponseBody
  public String hobbies(@RequestParam List<String> hobbies){
      System.out.println("bobbies: " + hobbies);
      return "bobbies: " + hobbies;
  }
  ```

  ```http
  http://localhost/employee/list?hobbies=sing&hobbies=sance&hobbies=rap&hobbies=basketball!
  ```

### 3.3 响应Json数据

1. 在pom中导入坐标

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

2. 在配置中开启json转对象的功能``@EnableWebMvc``

```java
@Configuration
@ComponentScan("com.example.controller")
@EnableWebMvc
public class SpringMvcConfig {

}
```

3. 编写Cotroller代码

```java
@RequestMapping("/json")
@ResponseBody
public String json(@RequestBody List<String> hobbies){
    System.out.println("bobbies: " + hobbies);
    return "hobbies: " + hobbies;
}
```

- Json数据传递引用类型参数

  ```java
  @RequestMapping("/jsonpojo")
  @ResponseBody
  public String jsonPojo(@RequestBody Employee emp){
      System.out.println("emp: " + emp);
      return "emp: " + emp;
  }
  ```

  ```json
  {
      "name": "harry",
      "age": 15
  }
  ```

  ```java
  @RequestMapping("/jsonpojolist")
  @ResponseBody
  public String jsonPojoList(@RequestBody List<Employee> emps){
      System.out.println("emps: " + emps);
      return "emps: " + emps;
  }
  ```

  ```json
  [
      {
          "name":"harry",
          "age":13
      },{
          "name":"potter",
          "age":25
      }
  ]
  ```

  

- RequestBody和@RequestParam区别
  - 区别
    - RequestParam用于url地址传参，表单传参[application/x-www-form-urlencoded]
    - @RequestBody用于接收json数据[application/json]
  - 应用
    - 后期开发中，发送json格式数据为主，@RequestBody应用较广
    - 如果发送非json格式数据，选用@RequestParam接收请求参数

### 3.4 日期类型参数传递

- 可以直接使用get在url传

```java
@RequestMapping("/date")
@ResponseBody
public String date(Date date){
    System.out.println("date: " + date);
    return "date: " + date;
}
```

```http
http://localhost/employee/date?date=2012/10/11
```

- 不同的日期格式：

```java
@RequestMapping("/date")
@ResponseBody
public String date(@DateTimeFormat(pattern="yyyy-MM-dd") Date date){
    System.out.println("date: " + date);
    return "date: " + date;
}
```

```http
http://localhost/employee/date?date=2012-10-11
```

- 加上时刻

```java
@RequestMapping("/date")
@ResponseBody
public String date(@DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss") Date date){
    System.out.println("date: " + date);
    return "date: " + date;
}
```

```http
http://localhost/employee/date?date=2012-10-11 10:08:08
```

### 3.5 页面跳转

不需要写@ResponseBody

```java
@Controller
public class indexController {
    @RequestMapping("/jump")
    public String toPage(){
        System.out.println("jump");
        return "index.jsp";
    }
}
```

### 3.6 响应POJO对象

```java
@RequestMapping("/responsepojo")
@ResponseBody
public Employee responsePojo(){
    Employee emp = new Employee();
    emp.setAge("100");
    emp.setName("zhangsan");
    System.out.println(emp);
    return emp;
}
```

# 四、REST风格

### 4.1 REST入门案例

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
public String update(@RequestBody User user);

// 查询指定
@RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
public String getById(@PathVariable Interger id);

//查询全部
@RequestMapping(value = "/users", method = RequestMethod.GET)
public String getAll();
```

- 设定http请求动作
- 设定请求变量（路径参数）

### 4.2 注解区别与应用

@RequestBody、@RequestParam、@PathVariable

- 区别
  - @RequestParam用于接收url地址传参或表单传参
  - @RequestBody用于接收json数据
  - @PathVariable用于接受路径参数，使用{参数名称}描述路径参数
- 应用
  - 后期开发中，发送的请求参数超过一个时，以json格式为主，@RequestBody应用较广
  - 如果发送非json格式参数，选用@RequestParam接受请求参数
  - 采用RESTful进行卡开发，当参数数量较少时，例如1个，可以采用@PathVariable接收参数，常用于传递id值

### 4.3 REST快速开发

- 方法路径相同可以合并标注至类上方
- 所有方法都有@ResponseBody，可以合并标注至类上方

- 使用@RestController可以合并表示@Controller和@ResponseBody
- ``@PostMapping``等价于``@RequestMapping(method = RequestMethod.POST)``

- ``@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)``等价于``@DeleteMapping("/{id}")``

```java
//@Controller
//@ResponseBody
@RestController
@RequestMapping("/simrest/users")
public class SimplifyRestController {
//    @RequestMapping(value = "/users", method = RequestMethod.POST)
//    @ResponseBody
    @PostMapping
    public String save(){
        System.out.println("user save");
        return "{'module':'user save'}";
    }

//    @RequestMapping(value = "/users/{id}", method = RequestMethod.DELETE)
//    @ResponseBody
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Integer id){
        System.out.println("user delete "+ id);
        return "{'module':'delete'}";
    }

//    @RequestMapping(value = "/users", method = RequestMethod.PUT)
//    @ResponseBody
    @PutMapping
    public String update(@RequestBody Employee emp){
        System.out.println("user update "+ emp);
        return "{'module':'update'}";
    }

//    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
//    @ResponseBody
    @GetMapping("/{id}")
    public String getById(@PathVariable Integer id){
        System.out.println("user getById "+ id);
        return "{'module':'getById'}";
    }

//    @RequestMapping(value = "/users", method = RequestMethod.GET)
//    @ResponseBody
    @GetMapping
    public String getAll(){
        System.out.println("user getAll ");
        return "{'module':'getAll'}";
    }
}
```

### 4.4 设置对静态资源的访问放行

1. 设置放行路径

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 当访问/pages/????的时候，走/pages目录下的内容
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }
}
```

2. 将该类设置为配置类，并被SpringMVC的控制类扫描

```java
@Configuration
@ComponentScan({"com.example.controller","com.example.config"})
@EnableWebMvc
public class SpringMvcConfig {

}
```

# 五、 SSM整合

### 5.1 整合流程

1. 创建工程

2. SSM整合
   - Spring
     - SpringConfig
   - MyBatis
     - MyBatisConfig
     - JdbcConfig
     - jdbc.properties
   - SpringMVC
     - ServletConfig
     - SpringMvcConfig
3. 功能模块
   - 表与实体类
   - dao（接口+自动代理）
   - service（接口+实现类）
     - 业务层测试接口（整合JUnit）
   - controller
     - 表现层接口测试（PostMan）

### 5.2 Jdbc写库中文乱码问题

```properties
jdbc.url=jdbc:mysql:///ssm_db?characterEncoding=utf8
```

### 5.3 异常处理器

- 出现异常的常见位置与常见诱因如下：
  - 框架内部抛出的异常：因使用不合规导致
  - 数据层抛出的异常：因外部服务器故障导致（例如：服务器访问超时）
  - 业务层抛出的异常：因业务逻辑书写错误导致（例如：遍历业务书写操作，导致索引异常等）
  - 表现层抛出的异常：因数据收集、校验等规则导致（例如：不匹配的数据类型导致的异常）
  - 工具类抛出的异常：因工具类书写不严谨不够健壮导致（例如：必要释放的连接长期未释放） 
- 所有的异常均抛出到表现层处理
- 异常处理器：集中地、统一地处理项目中出现的异常

```java
@RestControllerAdvice
public class ProjectExceptionAdvice {
    @ExceptionHandler(Exception.class)
    public Result doException(Exception e){
        System.out.println("有异常！");
        return new Result(404,null, "Server no reply!");
    }
}
```

要确保该类被配置类扫描到：

```java
@Configuration
@ComponentScan({"com.example.controller","com.example.config"}) // 包含了ProjectExceptionAdvice的包
@EnableWebMvc
public class SpringMvcConfig {

}
```

> 异常处理器
>
> 名称：@ExceptionHandler
>
> 类型：方法注解
>
> 位置：专用于异常处理的控制器方法上方
>
> 作用：设置指定异常的处理方案，功能等于控制器方法，出现异常后中止原始控制器执行，并转入当前方法执行
>
> 说明：此类方法可以根据处理的异常不同，制作多个方法分别处理对应的异常

### 5.4 异常分类与处理方法

异常分类：

- 业务异常（BusinessException）
  - 规范的用户行为产生的异常
  - 不规范的用户行为产生的异常
- 系统异常（SystemException）
  - 项目运行过程中可预计且无法避免的异常
- 其他异常（Exception）
  - 编程人员未预期到的异常

异常处理：

- 业务异常（BusinessException）
  - 发送对应消息传递给用户，提醒规范操作
- 系统异常（SystemException）
  - 发送固定消息传递给用户，安抚用户
  - 发送特定消息给运维人员，提醒维护
  - 记录日志
- 其他异常（Exception）
  - 发送固定消息传递给用户，安抚用户
  - 发送特定消息给运维人员，提醒维护
  - 记录日志

### 5.5 项目异常处理

1. 自定义项目系统级异常

```java
package com.example.exception;

public class SystemException extends RuntimeException{
    private Integer code;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public SystemException(Integer code) {
        this.code = code;
    }

    public SystemException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public SystemException(Integer code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
}
```

2. 自定义项目业务级异常

```java
package com.example.exception;

public class BusinessException extends RuntimeException{
    private Integer code;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public BusinessException(Integer code) {
        this.code = code;
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(Integer code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }
}
```

3. 自定义异常编码

```java
package com.example.controller;

public class Code {
    public static final Integer SAVE_OK = 20011;
    public static final Integer DELETE_OK = 20021;
    public static final Integer UPDATE_OK = 20031;
    public static final Integer GET_OK = 20041;

    public static final Integer SAVE_ERR = 20010;
    public static final Integer DELETE_ERR = 20020;
    public static final Integer UPDATE_ERR = 20030;
    public static final Integer GET_ERR = 20040;

    public static final Integer SYSTEM_ERR = 50001;
    public static final Integer SYSTEM_TIME_ERR = 50002;

    public static final Integer BUSINESS_ERR = 60002;
    public static final Integer SYSTEM_UNKOWN_ERROR = 59999;
}
```

4. 触发自定义异常

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    private BookDao bookDao;

    @Override
    public Book getById(Integer id) {
        if(id < 0){
            throw new BusinessException(Code.BUSINESS_ERR, "参数错误！");
        }
        return bookDao.getById(id);
    }
```

5. 拦截并处理异常

```java
package com.example.controller;

@RestControllerAdvice
public class ProjectExceptionAdvice {
    @ExceptionHandler(SystemException.class)
    public Result doSystemException(SystemException e){
        // 记录日志
        // 发送消息给运维
        // 发送消息给开发人员
        System.out.println("有异常！");
        return new Result(e.getCode(),null, e.getMessage());
    }

    @ExceptionHandler(BusinessException.class)
    public Result doBusinessException(BusinessException e){
        System.out.println("有异常！");
        return new Result(e.getCode(),null, e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result doException(Exception e){
        System.out.println("有异常！");
        return new Result(Code.SYSTEM_UNKOWN_ERROR,null,"系统繁忙，请稍后再试");
    }
}
```

# 六、 拦截器

### 6.1 拦截器概念

- 拦截器是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制方法的执行
- 作用：
  - 在指定的方法调用前后执行预先设定的代码
  - 组织原始方法的执行
- 拦截器与过滤器的区别：
  - 归属不同：Filter属于Servlet技术，Interceptor属于SpringMVC技术
  - 拦截内容不同：Filter对所有访问进行增强，Interceptor仅针对SpringMVC的访问进行增强
  - 

### 6.2 入门案例

1. 声明拦截器的bean，并实现HandlerInterceptor接口，并被SpringMVC配置类加载

```java
package com.example.controller.interceptor;

@Component
public class ProjectInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("PreHandler");
        return true; // return true表示放行，false则不会放行，直接结束访问
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandler");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

```java
@Configuration
@ComponentScan({"com.example.controller","com.example.config"})
@EnableWebMvc
public class SpringMvcConfig {

}
```

2. 定义配置类，继承WebMvcConfigurationSupport，实现addInterceptor方法
3. 添加拦截器并设定拦截的访问路径，路径可以通过可变参数设置多个

```java
package com.example.config;


@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Autowired
    private ProjectInterceptor projectInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books", "/books/*");
    }
}
```
*可以对SpringMvcConfig.java和SpringMvcSupport.java进行合并，简化开发（较强侵入式）

> |com
>
> |----example
>
> |--------config
>
> |------------SpringMvcConfig.java
>
> |------------SpringMvcSupport.java

合并到SpringMvcConfig.java中

```java
@Configuration
@ComponentScan({"com.example.controller"})
@EnableWebMvc
public class SpringMvcConfig implements WebMvcConfigurer{
	@Autowired
    private ProjectInterceptor projectInterceptor;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 当访问/pages/????的时候，走/pages目录下的内容
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(projectInterceptor).addPathPatterns("/books", "/books/*");
    }
}
```

- 拦截器执行顺序：
  - preHandle
    - return true
      - controller
      - postHandler
      - afterCompletion
    - return false
      - 结束

### 6.3 拦截器参数

- 前置处理

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    System.out.println("PreHandler");
    return true;  // return true表示放行，false则不会放行,直接结束流程
}
```

- 参数
  - request：请求对象
  - response：响应对象
  - handler：被调用的处理器对象，本质上是一个方法对象，对反射技术中的Method进行了再包装
- 返回值
  - 返回值为false，被拦截的处理器将不执行



- 后置处理

```java
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    System.out.println("postHandler");
}
```

- 参数
  - modelAndView：如果处理器执行完成具有返回结果，可以读取到对应数据与页面信息，并进行调整



- 完成后处理

```java
public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    System.out.println("afterCompletion");
}
```

- 参数
  - ex：如果处理器执行过程中出现异常对象，可以针对异常情况进行单独处理

### 6.4 拦截器工作流程分析

- 多拦截器执行顺序
  - 当配置多个拦截器时，形成拦截器链
  - 拦截器链的运行顺序参照拦截器添加顺序
  - 拦截器123的pre均返回true时：pre1->pre2->pre3->controller->post3->post2->post1->after3->after2->after1
  - 拦截器1、2为true，3为false：pre1->pre2->pre3->after2->after1
  - 拦截器1为true，2为false：pre1->pre2>after1
  - 拦截器1为false：pre1

