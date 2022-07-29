# 一、分模块开发与设计

### 1.1 分模块开发意义

- 将原始模块按照功能拆分成若干个子模块，方便模块间的相互调用，接口共享

### 1.2 分模块开发步骤

1. 创建maven模块

2. 书写模块代码

   分模块开发需要先针对模块功能进行设计，再进行编码

3. 通过maven指令安装模块到本地仓库（install指令）

   团队内部开发需要发布模块到团队内部可共享的仓库中（私服）

# 二、依赖管理

- 依赖指当前项目运行所需的jar，一个项目可以设置多个依赖
- 格式

```xml
<dependencies>
	<dependency>
    	<groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
</dependencies>
```

- 依赖具有传递性
  - 直接依赖：在当前项目中通过依赖配置建立的依赖关系
  - 间接依赖： 被资源的资源如果依赖其他资源，当前项目简介依赖其他资源
- 依赖冲突问题
  - 路径优先：当依赖中出现相同的资源时，层级越深，优先级越低，层级越浅，优先级越高
  - 声明优先：当资源在相同的层级被依赖时，配置顺序靠前的覆盖顺序靠后的
  - 特殊优先：当同级配置了相同资源的不同版本，后配置的覆盖前配置的

### 3.1 可选依赖
  可选依赖指对外隐藏当前依赖所依赖的资源——不透明

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.10.RELEASE</version>
      <!--可选依赖是隐藏当前工程所依赖的资源，隐藏后对应资源将不具有依赖传递性-->
      <optional>false</optional>
  </dependency>
  ```

### 3.2 排除依赖

  排除依赖指主动断开依赖的资源，被排除的资源无需指定版本

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.10.RELEASE</version>
      <!--排除依赖是指主动断开依赖的资源-->
      <exclusions>
      	<exclusion>
          	<groupId>log4j</groupId>
              <artifact>log4j</artifact>
          </exclusion>
          <exclusion>
          	<groupId>org.mybatis</groupId>
              <artifact>mybatis</artifact>
          </exclusion>
      </exclusions>
  </dependency>
  ```

  

# 三、继承与聚合

### 3.1 聚合

- 聚合：将多个模块组织成一个整体，同事进行项目构建的过程成为聚合
- 聚合工程：通常是一个不具有业务功能的“空”工程（有且仅有一个pom文件）
- 作用：使用聚合工程可以将多个工程编组，通过对聚合工程进行构建，实现对所包含的模块进行同步构建。当工程中某个模块发生更新（变更）时，必须保障工程中与已更新模块关联的模块同步更新，此时可以使用聚合工程来解决批量模块同步构建的问题。

### 3.2 聚合工程开发

1. 创建maven模块，设置打包类型为pom

   每个maven工程都有对应的打包方式，默认为jar，web工程打包方式为war

```xml
<packaging>pom</packaging>
```

2. 设置当前聚合工程所包含的子模块名称

   聚合工程中所包含的模块在进行构建时会根据模块间的依赖关系设置构建顺序，与聚合工程中模块的配置书写位置无关

   参与聚合的工程无法向上感知是否参与聚合，只能向下配置哪些模块参与本工程的聚合

```xml
<modules>
    <module>../maven_dao</module>
    <module>../maven_pojo</module>
    <module>../maven_ssm</module>
</modules>
```

### 3.3 继承

- 概念：继承描述的是两个工程之间的关系，与java中的继承类似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承
- 作用：简化配置、减少版本冲突

### 3.4 继承关系

1. 创建Maven模块，设置打包类型为pom

```xml
<packaging>pom</packaging>
```

2. 在父工程的pom文件中配置依赖关系（子工程沿用父工程中的依赖关系）

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.6</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.16</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.0</version>
    </dependency>

</dependencies>
```

3. 在父工程中配置子工程中可选的依赖关系

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

4. 在子工程中配置当前工程所继承的父工程

```xml
<parent>
    <groupId>com.example</groupId>
    <artifactId>maven_parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../maven_parent</relativePath>
</parent>
```

5. 在子工程中配置使用父工程中可选依赖的坐标

   子工程中使用父工程中的可选依赖，仅需要提供群组id和项目id，无需提供版本，版本由父工程统一提供，避免版本冲突。子工程中还可以定义父工程中没有定义的依赖关系

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.5 继承与聚合

- 作用
  - 聚合用于快速构建项目
  - 继承用于快速配置
- 相同点
  - 聚合与继承的pom文件打包方式均为pom，可以将两种关系只做到一个pom文件中
  - 聚合与继承属于设计型模块，并无实际的模块内容
- 不同点
  - 聚合是当前模块中配置关系，聚合可以感知到擦闽南语聚合的模块有哪些
  - 继承是在子模块中配置关系，父模块无法感知哪些子模块继承了自己

# 四、属性

### 4.1 属性配置与使用

1. 定义属性

```xml
<properties>
    <spring.version>5.2.10.RELEASE</spring.version>
</properties>
```

2. 引用属性

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
</dependency>
```

### 4.2 资源文件

1. 定义属性

```xml
<properties>
    <spring.version>5.2.10.RELEASE</spring.version>
    <jdbc.url>jdbc:mysql:///ssm_db?characterEncoding=utf8</jdbc.url>
</properties>
```

2. 配置文件中引用属性

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=${jdbc.url}
jdbc.username=root
jdbc.password=123456
```

3. 开启资源文件目录加载属性的过滤器

```xml
<build>
    <resources>
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

4. 配置maven打war包时，忽略web.xml检查

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.2</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```

### 4.3 maven属性

- 属性列表
  1. 自定义属性
  2. 内置属性
  3. Setting属性
  4. Java属性
  5. 环境变量属性

### 4.4 版本管理

- 工程版本
  - SNAPSHOT（快照版本）
    - 项目开发过程中临时输出的版本，称为快照版本
    - 快照版本会随着开发的进展不断更新
  - RELEASE（发布版本）
    - 项目开发进入到阶段里程碑之后，想团队外部发布稳定的版本，这种版本对应的构件文件时稳定的，即使进行功能的后续开发，也不会改变当前发布版本内容，这种版本称为发布版本
  - 发布版本
    - alpha版
    - beta版
    - 纯数字版

# 五、 多环境配置与应用

### 5.1 多环境开发

maven提供多种环境的设定，帮助开发者使用过程中快速切换环境

1. 定义多环境

```xml
<!--配置多环境-->
<profiles>
    <!--开发环境-->
    <profile>
        <id>env_dev</id>
        <properties>
            <jdbc.url>jdbc:mysql:///ssm_db?characterEncoding=utf8</jdbc.url>
        </properties>
    </profile>
    
    <!--生产环境-->
    <profile>
        <id>env_pro</id>
        <properties>
            <jdbc.url>jdbc:mysql://127.1.1.1:3306/ssm_db?characterEncoding=utf8</jdbc.url>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>-->
        </activation>
    </profile>
    
    <!--测试环境-->
    <profile>
        <id>env_test</id>
        <properties>
            <jdbc.url>jdbc:mysql://127.1.1.2:3306/ssm_db?characterEncoding=utf8</jdbc.url>
        </properties>
    </profile>
</profiles>
```

2. 使用多环境

```
mvn 指令 -P 环境定义id
```

例如：

```
mvn install -P env_dev
```

### 5.2 跳过测试

执行的项目构建指令必须包含测试生命周期，否则无效果。例如执行compile生命周期，不经过test生命周期

```
mvn 指令 -D skipTests
```

```
mvn install -D skipTests
```

- 细粒度控制跳过测试

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.12.4</version>
        <configuration>
            <skipTests>false</skipTests>
            <!--排除不参与测试的内容-->
            <excludes>
                <exclude>**/BookServiceTest.java</exclude>
            </excludes>
        </configuration>
    </plugin>
</plugins>
```

# 六、私服

私服简介

- 私服是一台独立的服务器，用于解决团队内部的资源共享与资源同步问题

- Nexus：Snatype公司的一款maven私服产品

### 6.1 Nexus安装与启动

- 启动服务器

```cmd
nexus.exe /run nexus
```

- 访问服务器

```html
http://localhost:8081
```

- 修改基本配置信息
  - 安装路径下etc目录中nexus-default.properties文件保存有nexus基础配置信息，例如默认访问端口
- 修改服务器运行配置信息
  - 安装路径下bin目录中nexus.vmoptions文件保存有nexus服务器启动对应的配置信息，例如默认占用空间

### 6.2 私服仓库类别

| 仓库类别 | 英文名称 |          功能           | 关联操作 |
| :------: | :------: | :---------------------: | :------: |
| 宿主仓库 |  hosted  | 保存自主研发+第三方资源 |   上传   |
| 代理仓库 |  proxy   |    代理连接中央仓库     |   下载   |
|  仓库组  |  group   |  为仓库组简化下载操作   |   下载   |

### 6.3 资源的上传与下载

1. 配置位置（setting.xml文件中）

```xml
<mirrors>
	<mirror>
    	<id>nexus-xjtu</id>
        <mirrorOf>*</mirrorOf>
        <url>http://localhost:8081/repository/maven-public/</url>
    </mirror>
</mirrors>
```

2. 配置位置（工程pom文件中）

```xml
<distributionManagement>
	<repository>
    	<id>xjtu-release</id>
        <url>http://localhost:8081/repository/xjtu-release</url>
    </repository>
    <snapshotRepository>
    	<id>xjtu-snapshot</id>
        <url>http://localhost:8081/repository/xjtu-snapshot</url>
    </repository>
</distributionManagement>
```

发布命令

```
mvn deploy
```

