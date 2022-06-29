### 一、入门案例

- 新建空工程
- Project Structure -> Project Settings -> Modules -> + ->Spring Initializer -> 选择SDK填写工程信息 -> 选择SpringBoot版本 -> 勾选Spring Web
- 若spring网站的访问速度低下，可以选择阿里镜像源：https://start.aliyun.com，工程模板略有不同



### 二、SpringBoot简介

为了简化Spring应用的初始搭建和开发过程

- Spring程序缺点

  - 依赖设置繁琐
  - 配置繁琐

- SpringBoot有程序优点

  - 起步依赖（简化依赖配置）
  - 自动配置（简化常用工程相关配置）
  - 辅助功能（内置服务器）

- 入门案例解析

  - starter

    SpringBoot常见项目名称，定义了当前项目使用的所有依赖坐标，以达到减小配置以来的的目的

  - parent

    所有SpringBoot项目要继承的项目，定义了若干个版本坐标号（依赖管理，而非依赖），以达到减少使用依赖冲突的目的

    spring-boot-starter-parent各版本之间存在着诸多坐标版本不同，使用坐标时，仅书写G(roupid) A(rtifactid) V(ersion)的GA，V由SpringBoot提供，除非SpringBoot未提供对应版本。

  - 引导类

    - SpringBoot的引导类是Boot工程的执行入口，运行main方法就可以启动项目
    - SpringBoot工程运行后初始化Spring容器，扫描引导类所在包加载bean
    
  - 内嵌Tomcat

### 三、REST风格

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