*本笔记为转载，原链接[稀土掘金](https://juejin.cn/post/6844903822687485965)，有所增改。

# 一、简介

- slf4j：一个针对各类Java日志框架的统一facade抽象。

- java常见日志框架：java.util.logging, log4j, logback, commons-logging

- logback是log4j的作者开发的新一代日志框架，目前应用最广泛。__SpringBoot默认使用logback，默认INFO级别。__logback日志加载顺序：logback.xml -> application.properties -> logback-spring.xml

# 二、日志级别

log4j定义的日志级别：debug/info/warn/error/fatal

warn，潜在错误；error，错误，可能导致程序退出；fatal，严重错误，程序会退出

还有两个特殊的级别：OFF-最高级别，ALL-最低级别

log4j建议仅使用__debug/info/warn/error__四个级别

日志级别：ERROR -> WARN -> INFO -> DEBUG

如配置日志级别为INFO，则INFO及以上级别的日志会输出，而比INFO级别低的日志（debug日志）不会被输出。

# 三、POM导入包

直接引入：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
  <version>2.1.11.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

__间接引入（SpringBoot自带）__：

引入spring-boot-starter，会自动引入spring-boot-starter-logging

引入spring-boot-starter-web，会自动引入spring-boot-starter

# 四、通过springboot配置文件配置logback

在``application.yml``中配置logback，使日志输出日志到文件，并可对不同包下的日志设置不同的过滤级别：

```yml
logging:
  file:
    name: logback-demo.log
  level:
    com.example.service: debug
```

日志默认是叠加输出，即每次启动项目不会删除之前的日志文件，也不会将当前使用的日志文件清空，而是在下面另起一行。

# 五、在项目中使用logger

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    private static final Logger LOGGER = LoggerFactory.getLogger(UserService.class);

    public IPage<User> getById(Page page, int id){
        LOGGER.info("log info");
        LOGGER.debug("id: {}, pageSize: {}", id, page.getSize());
        return userMapper.getById(page, id);
    }
}
```

运行测试类或者通过Controller调用userService.getById()时，可以在日志文件``logback-demo.log``中查询到日志信息。

# 六、通过XMl文件自定义logback配置

## 6.1 日志配置简介

- 日志的设置与功能
  - 设置日志格式
  - 设置日志的大小
  - 分类保存日志
  - ...
- 日志框架默认配置文件
  - Logback：``logback-spring.xml``, ``logback-spring.groovy``,`` logback.xml``, ``logback.groovy``
  - Log4j：``log4j-spring.properties``, ``log4j-spring.xml``, ``log4j.properties``, ``log4j.xml``
  - Log4j2：``log4j2-spring.xml``, ``log4j2.xml``
  - JDK (Java Util Logging)：``logging.properties``
- 日志加载顺序：logback.xml -> application.properties -> logback-spring.xml

- __logback框架下建议使用logback-spring.xml，也可以在application中通过logging.config=classpath:xxx.xml来指定配置文件。__

## 6.2 具体设置项

- xml配置示例

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
      <contextName>logback-spring-demo-dev</contextName>
      <property name="pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg %n"/>
      <property name="pattern-color" value="%yellow(%d{yyyy-MM-dd HH:mm:ss.SSS}) [%thread] %highlight(%-5level) %green(%logger{50}) - %highlight(%msg) %n"/>
      <property name="LOG_HOME" value="logs"/>
  
      <!-- 控制台输出 -->
      <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
          <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
              <pattern>${pattern}</pattern>
          </encoder>
      </appender>
  
      <!-- 控制台输出-带颜色 -->
      <appender name="CONSOLE-WITH-COLOR" class="ch.qos.logback.core.ConsoleAppender">
          <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
              <pattern>${pattern-color}</pattern>
          </encoder>
      </appender>
  
      <!-- 文件输出 -->
      <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${LOG_HOME}/all.%d.%i.log</fileNamePattern>
              <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                  <maxFileSize>10MB</maxFileSize>
              </timeBasedFileNamingAndTriggeringPolicy>
              <maxHistory>30</maxHistory>
          </rollingPolicy>
  
          <encoder>
              <pattern>${pattern}</pattern>
          </encoder>
      </appender>
  
  
      <root level="INFO">
          <appender-ref ref="CONSOLE-WITH-COLOR"/>
          <appender-ref ref="FILE"/>
      </root>
  
      <logger name="com.example.logbackdemo.IndexAction" level="info" additivity="false">
          <appender-ref ref="CONSOLE"/>
      </logger>
  
  </configuration>
  ```

- 配置详解

  - 下面具体描述一下logback.xml中的配置项：两种属性``contextName``和``property``，三个节点``appender``、``root``、``logger``
  
    - contextName属性
  
      日志名，可以使用%contextName来引用。如果同时存在logback.xml和logback-spring.xml，或者同时存在logback.xml和自定义的配置文件，则会先加载logback.xml，再根据application配置加载指定配置文件，或加载logback-spring,xml。如果这两个配置文件的contextName不同，就会报错：
  
      ``ERROR in ch.qos.logback.classic.joran.action.ContextNameAction - Failed to rename context [logback-demo] as [logback-spring-demo-dev] java.lang.IllegalStateException: Context has been already given a name``
  
    - property属性
  
      property标签可用于自定义属性，比如定义一个LOG_HOME，然后使用${LOG_HOME}去引用它
  
    - appender节点
  
      appender的意思是追加器，在这里可以理解为一个日志的渲染器。比如渲染console日志为某种格式，渲染文件日志为另一种格式。appender中有``name``和``class``两个属性，有``rollingPolicy``和``encoder``两个子节点。name表示该渲染器的名字，class表示使用的输出策略，常见的有控制台输出策略和文件输出策略。
    
  - 控制台输出appender
  
    ```xml
    <property name="pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg %n"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    	<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      	<pattern>${pattern}</pattern>
      </encoder>
    </appender>
    ```
  
    encoder表示输出格式，具体说明如下：
  
    - %d表示时间
    - %thread表示线程名
    - %-5level表示日志级别，允许以五个字符长度输出
    - %logger{50}表示日志的输出者，比如类名
    - %msg表示具体的日志消息，就是logger.info("xxx")中的xxx
    - %n表示换行
  
    还可以定义颜色和高亮：
  
    ```xml
    <property name="pattern-color" value="%yellow(%d{yyyy-MM-dd HH:mm:ss.SSS}) [%thread] %highlight(%-5level) %green(%logger{50}) - %highlight(%msg) %n"/>
    ```
  
    即使用%color(xxx)来指定对应列的输出颜色，可以实现控制台输出的颜色高亮。
  
  - 文件输出appender
  
    文件输出主要包括配置：以指定格式将日志输出到指定文件夹下的文件中，可以配置该文件的名称、最大大小、保存时间
  
    ```xml
    <property name="LOG_HOME" value="logs"/>
    <property name="pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg %n"/>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_HOME}/all.%d.%i.log</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        	<maxFileSize>10MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
        <maxHistory>30</maxHistory>
      </rollingPolicy>
    
      <encoder>
        <pattern>${pattern}</pattern>
        </encoder>
    </appender>
    ```
  
    上述配置的主要内容是：以指定的格式向logs文件下的文件输出日志，文件名称格式被指定为logs/all.日期.索引号.log，日志文件最大大小为10MB，超出则创建新文件，日志文件保留30天。索引从0开始递增，rollingPolicy指滚动策略，关于日志的文件大小限制也可以使用1GB.
  
  - root节点-启用配置
  
    root节点实际上是配置启用哪种appender，可以添加多个appender
  
    ```xml
    <root level="INFO">
    	<appender-ref ref="CONSOLE-WITH-COLOR"/>
      <appender-ref ref="FILE"/>
    </root>
    ```
  
    表示level是info级别，启用渲染器CONSOLE-WITH-COLOR和FILE。
  
    按照这样的配置，输出日志时，控制台会按照CONSOLE定义的格式输出，而日志文件会按照CONSOLE-WITH-COLOR的配置去输出。
  
  - logger节点
  
    对单个包或类添加配置，覆盖上面的root
  
    ```xml
        <!-- logger节点可以指定具体包或类的日志配置 -->
        <!-- name属性为必选，指定要配置的包或类，level和additivity为可选，有缺省值 -->
        <!-- level表示日志级别，这里配置info级别，表示info及以上级别的日志被输出 -->
        <!-- additivity表示日志是否传递到上一级，默认为true -->
        <logger name="com.example.logbackdemo.IndexAction" level="info" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
    ```

# 七、指定启用哪些日志配置

1. SpringBoot指定启用哪个xml

   我们可以在springboot的配置文件中指明使用哪个xml作用logger的配置文件，比如：

   ``logging.config=classpath:logback-spring-dev.xml``

   如果不指定，则会按照logback.xml -> application.yml -> logback-spring.xml的顺序去查找是否存在默认的配置。

2. xml中使用springProfile标签指定哪些环境下应该使用哪些配置

   我们可以在具体的日志配置xml中指定哪些环境下使用哪些配置，比如：

   ```xml
   <springProfile name="local,dev">
   	<root level="INFO">
   		<appender-ref ref="CONSOLE-WITH-COLOR"/>
   		<appender-ref ref="FILE"/>
   	</root>
   </springProfile>
   
   <springProfile name="prod">
   	<root level="INFO">
   		<appender-ref ref="CONSOLE-WITH-COLOR"/>
   		<appender-ref ref="FILE"/>
   	</root>
   </springProfile>
   ```

   这样的话，就不需要建立多个xml文件了，使用一个默认的logback-spring.xml即可。