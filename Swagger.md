# 一、Swagger简介

- Swagger是一款Restful接口的文档在线自动生成和功能测试功能软件。
- Swagger是一个规范和完整的框架，用于生成、描述、调用和可视化Restful风格的Web服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。

# 二、Swagger优缺点

- 优点

  1. 节省了大量手写接口文档的时间

  2. 通过注解自动生成在线文档

  3. 接口在线调用调试

- 缺点
  1. 代码耦合，需要注解支持
  2. 代码侵入性比较强
  3. 无法测试错误的请求方式、参数及不限于这些

- 手动API痛点
  1. 文档更新的时候，需要再次发送一份给前端，也就是文档更新交流不及时
  2. 接口返回结果不明确
  3. 不能直接在线测试接口，通常需要使用工具，比如postman
  4. 接口文档太多，不好管理

# 三、SpringBoot整合Swagger

###  3.1 引入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 3.2 书写Swagger配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                // 指定构建api文档的详细信息的方法：apiInfo()
                .apiInfo(apiInfo())
                .select()
                // 指定要生成api接口的包路径
                .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
                //使用了 @ApiOperation 注解的方法生成api接口文档
                //.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                //可以根据url路径设置哪些请求加入文档，忽略哪些请求
                .build();
    }

    /**
     * 设置api文档的详细信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 标题
                .title("Spring Boot & Swagger2")
                // 接口描述
                .description("swagger")
                // url
                .termsOfServiceUrl("")
                // 版本信息
                .version("1.0")
                // 构建
                .build();
    }
}
```

**@Configuration** 标注在类上，相当于把该类作为spring的xml配置文件中的，作用为：配置spring容器(应用上下文)。用@Bean标注方法等价于XML中配置bean。 **@EnableSwagger2** 的作用是启用Swagger2相关功能。 **Docket对象包含三个方面信息**：

1. 整个API的描述信息，即ApiInfo对象包括的信息，这部分信息会在页面上展示。
2. 指定生成API文档的包名。
3. 指定生成API的路径。

### 3.3 在controller类/pojo类中添加描述信息

```java
@Api("书籍相关的Controller")
@RestController
@RequestMapping("/books")
public class BookController {
    @Autowired
    private BookService bookService;

    @ApiOperation("保存操作")
    @PostMapping
    public Result save(@RequestBody Book book){
        boolean flag =  bookService.save(book);
        return new Result(flag? Code.SAVE_OK : Code.SAVE_ERR, flag);
    }

    @ApiOperation("根据ID删除")
    @DeleteMapping("/{id}")
    public Result delete(@PathVariable Integer id){
        boolean flag = bookService.delete(id);
        return new Result(flag? Code.DELETE_OK : Code.DELETE_ERR, flag);
    }

    @ApiOperation("根据ID修改")
    @PutMapping
    public Result update(@RequestBody Book book){
        boolean flag = bookService.update(book);
        return new Result(flag? Code.UPDATE_OK : Code.UPDATE_ERR, flag);
    }

    @ApiOperation("根据ID查询")
    @GetMapping("/{id}")
    public Result getById(@PathVariable Integer id){
        Book book = bookService.getById(id);
        Integer code = book != null ? Code.GET_OK : Code.GET_ERR;
        String msg = book != null ? "" : "查询失败，请重试！";
        return new Result(code, book, msg);
    }

    @ApiOperation("查询全部")
    @GetMapping
    public Result getAll(){
        List<Book> books = bookService.getAll();
        Integer code = books != null ? Code.GET_OK : Code.GET_ERR;
        String msg = books != null ? "" : "查询失败，请重试！";
        return new Result(code, books, msg);
    }

    @ApiOperation("调试获取错误")
    @RequestMapping("/err")
    public Result getErr(){
        bookService.transaction();
        return new Result(11111,null, null);
    }
}
```

```java
@Mapper
@ApiModel
public class Book {
    @ApiModelProperty("用户id")
    private Integer id;
    @ApiModelProperty("类型")
    private String type;
    @ApiModelProperty("用户名")
    private String name;
    @ApiModelProperty("用户描述")
    private String description;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", type='" + type + '\'' +
                ", name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
```

- 常用注解说明：

**@Api**:用于controller类上，说明该类的作用

```sql
tags=“说明该类的作用，可以在UI界面上看到的注解”
value=“该参数没什么意义，在UI界面上也看到，所以不需要配置”
description = "描述"
```

**@ApiOperation**：用在controller的方法上，用来说明方法用途、作用

```sql
value=“说明方法的用途、作用”
notes=“方法的备注说明”
```

**@ApiImplicitParam**：用来给方法入参增加说明

```sql
name：参数名
value：参数的汉字说明、解释
dataType： 参数类型，默认String
required ： 参数是否必传，true必传
defaultValue：参数的默认值
paramType：参数放在哪个地方
header请求参数的获取:@RequestHeader,参数从请求头获取
query请求参数的获取:@RequestParam,参数从地址栏问号后面的参数获取
path（用于restful接口）请求参数的获取：@PathVariable，参数从URL地址上获取
body（不常用）参数从请求体中获取
form（不常用）参数从form表单中获取
```

**@ApiImplicitParams：**用在controller的方法上，一组@ApiImplicitParam   **@ApiResponse：**用在 @ApiResponses里边，一般用于表达一个错误的响应信息

```sql
code：数字，例如400
message：信息，例如"请求参数没填好"
response：抛出异常的类
```

**@ApiResponses：** 用在controller的方法上，用于表示一组响应

__@ApiModel：__用在返回对象类上，描述一个Model的信息（这种一般用在Post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候） **@ApiModelProperty：** 用在出入参数对象的字段上，表示对model属性的说明或者数据操作更改

```sql
value = 字段说明
name = 重写属性名字
dataType = 重写属性类型
required = 是否必填，true必填
example = 举例说明
hidden = 隐藏
```

**@ApiIgnore：** 使用该注解忽略这个API，不会生成接口文档。可注解在类和方法上

### 3.4 swagger访问页面

启动项目后，输入[http://localhost:80/swagger-ui.html](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A8080%2Fswagger-ui.html%E8%AE%BF%E9%97%AE%EF%BC%8C%E9%A1%B5%E9%9D%A2%E5%A6%82%E4%B8%8B%EF%BC%9A)访问

### 3.5 可能的问题

- 如果paramType与方法参数获取使用的注解不一致，会直接影响到参数的接收。
- 控制层中定义的方法必须在@RequestMapper中显示的指定RequestMethod类型(get/post等)，否则SawggerUi会默认为全类型皆可访问， API列表中会生成多条项目。

- swagger的页面其实是由json数据组成，可以将这些json数据存储到数据库中，以便在不启动项目的情况下访问swagger。 [http://localhost:80/json/v2/api-docs](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A8010%2Fjson%2Fv2%2Fapi-docs)

