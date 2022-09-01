# 一、基本概念

__认证__：用户认证就是判断一个用户的身份是否合法的过程，用户去访问系统资源时系统要求验证用户的身份信息，身份合法方可继续访问，不合法则拒绝访问。

__授权__：根据用户的权限来控制用户访问资源的过程。

__OAuth2.0__：认证协议，用于分布式系统

__会话__：认证后为避免用户每次操作重复认证，将用户信息保存在会话中，会话是系统为了保持当前用户登陆状态所提供的机制。常见的机制有session和token。

### 1.1 基于session的认证方式

用户认证成功后，将用户相关数据保存在session中，将sesssion_id存放到cookie中，客户端请求时带上session_id就可以验证服务器端是否存在session数据，以此完成用户的合法校验，当用户退出系统或session过期销毁时，客户端的session_id也失效。

### 1.2 基于token的认证方式

用户认证成功后，服务端生成token发给客户端，客户端可以放到cookie或localStorage等存储中，每次请求时带上token，服务端收到token通过验证后即可确认用户身份。

### 1.3 授权的数据模型

Who对What进行How操作

Who：主体，可以是用户或程序，需要访问系统的资源

What：资源，如菜单、页面按钮，系统商品信息，函数方法等

How：权限/许可，查询权限添加权限、调用权限等

### 1.4 RBAC 

- RABC (Role Based Access Control)基于角色的访问控制是按照角色进行授权，例如主体的角色为总经理，可以查询企业运营报表，查询员工信息等。

- RBAC (Resource Based Access Controll)根据资源或权限进行授权，例如需要具有查询工资的权限

### 1.5 SpringSecurity工作原理

- 结构总览

  SpringSecurity所解决的问题是__安全访问控制__。而安全访问控制功能是指对所有进入系统的请求进行拦截，校验每个请求是否能够访问它期望的资源。可以通过Filter或者AOP技术实现，SpringSecurity是靠Filter实现的。

  当初始化SpringSecurity时，会创建一个名为SpringSecurityFilterChain的Servlet过滤器，类型为``org.springframework.security.web.FilterChainProxy``，它实现了``javax.servlet.Filter``，因此外部的请求会经过此类。

  ![架构图](https://miro.medium.com/max/1400/1*RiktfvkIuy955z5IgcqJWw.png)

  FilterChainProxy是一个代理，真正起作用的是FilterChainProxy中SecurityFilterChain所包含的各个Filter，同时这些Filter作为Bean直接被Spring管理，是SpringSecurity的核心。它们__不直接处理用户的认证或授权__，而是交给AuthenticationManager和AccessDecisionManager处理。

  ![结构图](https://miro.medium.com/max/1400/1*1wACv6k81-_RS-y3tGcrJw.png)

![授权](https://pic2.zhimg.com/v2-e3d2adb839bd6259b13d2004f7b465bd_r.jpg)

# 二、SpringBoot整合SpringSecurity

### 2.1 导入依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
~~~

### 2.2 编写Controller代码

```java
@RestController
@RequestMapping("/user")
public class FunctionController {
    @GetMapping("/findAll")
    public String findAll(){
        return "GET ALL";
    }
    
    @GetMapping("getMyInfo")
    public String getMyInfo(){
        return "USER INFO";
    }
}
```

### 2.3 启动访问接口

 ~~~java
//启动会在控制台出现
Using generated security password: 98687887-01bf-41b5-bb6e-63009367be0f
 ~~~

同时访问http://localhost/user/findAll ,会出现一个登陆页面

这时候，用户名输入user，密码输入上方控制台打印的密码，即可登录，并且正常访问接口。

### 2.4 登录的用户名/密码

对登录的用户名/密码进行配置，有三种不同的方式：

1. 在 application.yaml 中进行配置

   ~~~yaml
   spring:
     security:
       user:
         name: admin
         password: 123456
   
   server:
     port: 80
   ~~~

2. 通过 Java 代码配置在内存中

   ~~~java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           //下面这两行配置表示在内存中配置了两个用户
           auth.inMemoryAuthentication()
                   .withUser("admin").roles("admin").password("123456")
                   .and()
                   .withUser("user").roles("user").password("123456");
       }
       
       @Bean
       PasswordEncoder passwordEncoder() {
           return NoOpPasswordEncoder.getInstance();
       }
   
   //    public static void main(String[] args) {
   //        String mszlu = new BCryptPasswordEncoder().encode("123456");
   //        System.out.println(mszlu);
   //    }
   
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests()
                   .antMatchers("/user/getAll").hasRole("admin")
                   .anyRequest().authenticated()
                   .and().formLogin().permitAll()
                   .and().logout().permitAll();
       }
   }
   ~~~

### 2.5 使用自定义的登陆页面

1. 自定义登陆页面``login.html``

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Document</title>
   </head>
   <body>
   <form action="/login" method="post">
       用户名：<input type="text" name="userName"></input> <br/>
       密码：<input type="password" name="password" ></input><br/>
       <input type="submit" value="登录"/>
   </form>
   </body>
   </html>
   ```

2.  设置配置文件``SecurityConfig.java``

   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           //下面这两行配置表示在内存中配置了两个用户
           auth.inMemoryAuthentication()
                   .withUser("admin").roles("admin").password("123456")
                   .and()
                   .withUser("user").roles("user").password("123456");
       }
       @Bean
       PasswordEncoder passwordEncoder() {
           return NoOpPasswordEncoder.getInstance();
       }
   
   //    public static void main(String[] args) {
   //        String psw = new BCryptPasswordEncoder().encode("123456");
   //        System.out.println(psw);
   //    }
   
   
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests() //开启登录认证
                   .antMatchers("/user/findAll").hasRole("admin") //访问接口需要admin的角色
                   .antMatchers("/login").permitAll()
                   .anyRequest().authenticated() // 其他所有的请求 只需要登录即可
                   .and().formLogin()
                   .loginPage("/login.html") //自定义的登录页面
                   .loginProcessingUrl("/login") //登录处理接口
                   .usernameParameter("userName") //定义登录时的用户名的key 默认为username
                   .passwordParameter("password") //定义登录时的密码key，默认是password
                   .successHandler(new AuthenticationSuccessHandler() {
                       @Override
                       public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                           httpServletResponse.setContentType("application/json;charset=utf-8");
                           PrintWriter out = httpServletResponse.getWriter();
                           out.write("success");
                           out.flush();
                       }
                   }) //登录成功处理器
                   .failureHandler(new AuthenticationFailureHandler() {
                       @Override
                       public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException, IOException {
                           httpServletResponse.setContentType("application/json;charset=utf-8");
                           PrintWriter out = httpServletResponse.getWriter();
                           out.write("fail");
                           out.flush();
                       }
                   }) //登录失败处理器
                   .permitAll() //通过 不拦截，更加前面配的路径决定，这是指和登录表单相关的接口 都通过
                   .and().logout() //退出登录配置
                   .logoutUrl("/logout") //退出登录接口
                   .logoutSuccessHandler(new LogoutSuccessHandler() {
                       @Override
                       public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                           httpServletResponse.setContentType("application/json;charset=utf-8");
                           PrintWriter out = httpServletResponse.getWriter();
                           out.write("logout success");
                           out.flush();
                       }
                   }) //退出登录成功 处理器
                   .permitAll() //退出登录的接口放行
                   .and()
                   .httpBasic()
                   .and()
                   .csrf().disable(); //csrf关闭 如果自定义登录 需要关闭
       }
   }
   ```

# 三、数据库访问认证和授权

### 3.1 数据库访问认证

1. 导入依赖

   ```xml
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
   
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-security</artifactId>
       </dependency>
   
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
       </dependency>
   
       <dependency>
           <groupId>com.baomidou</groupId>
           <artifactId>mybatis-plus-boot-starter</artifactId>
           <version>3.4.1</version>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>druid</artifactId>
           <version>1.1.6</version>
       </dependency>
   
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <scope>runtime</scope>
       </dependency>
   </dependencies>
   ```
   
2. 创建用户密码表

   ~~~sql
   CREATE TABLE `admin_user`  (
     `id` bigint(0) NOT NULL AUTO_INCREMENT,
     `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `create_time` bigint(0) NOT NULL,
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
   
   INSERT INTO `admin_user`(`id`, `username`, `password`, `create_time`) VALUES (1, 'admin', '123456', 1622711132975);
   ~~~

3. 在项目yaml配置文件中配置数据库

   ```yaml
   spring:
     datasource:
       username: root
       password: 123456
       url: jdbc:mysql://localhost:3306/mybatisplus_db?serverTimezone=UTC
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
     main:
       banner-mode: off
   
   server:
     port: 80
   ```

4. 创建Java的pojo类

   ~~~java
   @Data
   public class AdminUser {
   
       private  Long id;
   
       private String username;
   
       private String password;
   
       private Long createTime;
   }
   ~~~

5. 创建User表对应Mapper

   ~~~java
   @Mapper
   public interface AdminUserMapper extends BaseMapper<AdminUser> {
   }
   ~~~

6. 创建Service类实现UserDetailService接口

   ~~~java
   @Component
   public class SecurityUserService implements UserDetailsService {
       @Autowired
       private AdminUserMapper adminUserMapper;
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           LambdaQueryWrapper<AdminUser> queryWrapper = new LambdaQueryWrapper<>();
           queryWrapper.eq(AdminUser::getUsername,username).last("limit 1");
           AdminUser adminUser = this.adminUserMapper.selectOne(queryWrapper);
           if (adminUser == null){
               throw new UsernameNotFoundException("用户名不存在");
           }
           List<GrantedAuthority> authorityList = new ArrayList<>();
           UserDetails userDetails = new User(username,adminUser.getPassword(),authorityList);
           return userDetails;
       }
   }
   ~~~

7. 删除原先配置文件中的的用户名密码信息

   ```java
   //    @Override
   //    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   //        //下面这两行配置表示在内存中配置了两个用户
   //        auth.inMemoryAuthentication()
   //                .withUser("admin").roles("admin").password("123456")
   //                .and()
   //                .withUser("user").roles("user").password("123456");
   //    }
   ```

8. __（可选）__在配置中添加使用UserDetailService

   当配置了不止一个登录的Service之后，需要在配置类``SecurityConfig.java``中指定使用哪一个，若只有一个实现了UserDetailService的类bean，则可以忽略该步骤。

   ~~~java
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.userDetailsService(securityUserService);
           //...
       }
   ~~~


### 3.2 数据库访问授权

1. 权限相关表结构及pojo类

   ~~~sql
   DROP TABLE IF EXISTS `role`;
   CREATE TABLE `role`  (
     `id` int(0) NOT NULL AUTO_INCREMENT,
     `role_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `role_desc` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `role_keyword` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of role
   -- ----------------------------
   INSERT INTO `role` VALUES (1, '管理员', '管理员', 'ADMIN');
   INSERT INTO `role` VALUES (2, '运营', '运营部门', 'BUSINESS');
   
   DROP TABLE IF EXISTS `user_role`;
   CREATE TABLE `user_role`  (
     `id` bigint(0) NOT NULL AUTO_INCREMENT,
     `user_id` bigint(0) NOT NULL,
     `role_id` int(0) NOT NULL,
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `user_id`(`user_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of user_role
   -- ----------------------------
   INSERT INTO `user_role` VALUES (1, 1, 1);
   ~~~

   ~~~java
   @Data
   public class Role {
       private Integer id;
   
       private String roleName;
   
       private String roleDesc;
   
       private String roleKeyword;
   }
   ~~~
   ~~~sql
   DROP TABLE IF EXISTS `permission`;
CREATE TABLE `permission`  (
    `id` int(0) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `desc` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `permission_keyword` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     `path` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of permission
   -- ----------------------------
   INSERT INTO `permission` VALUES (1, '查询全部', '查询全部', 'USER_FINDALL', '/user/findAll');
   INSERT INTO `permission` VALUES (2, '年龄查询', '年龄查询', 'USER_FINDAGE', '/user/findAge');
   
   DROP TABLE IF EXISTS `role_permission`;
   CREATE TABLE `role_permission`  (
     `id` bigint(0) NOT NULL AUTO_INCREMENT,
     `role_id` int(0) NOT NULL,
     `permission_id` int(0) NOT NULL,
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `role_id`(`role_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Records of role_permission
   -- ----------------------------
   INSERT INTO `role_permission` VALUES (1, 1, 1);
   ~~~
   ~~~java
   @Data
   public class Permission {
       private Integer id;
       private String name;
       private String desc;
       private String permissionKeyword;
       private String path;
   }
   ~~~
   
2. 创建对应mapper

   ```java
   @Mapper
   public interface PermissionMapper extends BaseMapper<Permission> {
       List<Permission> findPermissionByRole(Integer roleId);
   }
   ```
   
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!--MyBatis配置文件-->
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.example.mapper.PermissionMapper">
    <resultMap id="perMap" type="com.example.domain.Permission">
           <id column="id" property="id"/>
           <result column="name" property="name"/>
           <result column="desc" property="desc"/>
           <result column="permission_keyword" property="permissionKeyword"/>
           <result column="path" property="path"/>
       </resultMap>
   
       <select id="findPermissionByRole" parameterType="int" resultMap="perMap">
           select * from permission where id in (select permission_id from role_permission where role_id=#{roleId})
       </select>
   </mapper>
   ```
   
   ```java
   @Mapper
   public interface RoleMapper extends BaseMapper<Role> {
       List<Role> findRoleListByUserId(Long userId);
   }
   ```
   
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!--MyBatis配置文件-->
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.example.mapper.RoleMapper">
       <resultMap id="roleMap" type="com.example.domain.Role">
           <id column="id" property="id"/>
           <result column="role_name" property="roleName"/>
           <result column="role_desc" property="roleDesc"/>
           <result column="role_keyword" property="roleKeyword"/>
       </resultMap>
   
       <select id="findRoleListByUserId" parameterType="long" resultMap="roleMap">
           select * from role where id in (select role_id from user_role where user_id=#{userId})
       </select>
   </mapper>
   ```
   
3. 在UserDetailService的接口实现中，查询用户的权限

   ~~~java
   @Component
   public class SecurityUserService implements UserDetailsService {
       @Autowired
       private AdminUserMapper adminUserMapper;
       @Autowired
       private RoleMapper roleMapper;
       @Autowired
       private PermissionMapper permissionMapper;
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           LambdaQueryWrapper<AdminUser> queryWrapper = new LambdaQueryWrapper<>();
           queryWrapper.eq(AdminUser::getUsername,username).last("limit 1");
           AdminUser adminUser = this.adminUserMapper.selectOne(queryWrapper);
           if (adminUser == null){
               throw new UsernameNotFoundException("用户名不存在");
           }
           List<GrantedAuthority> authorityList = new ArrayList<>();
           //查询角色和角色对应的权限 并赋予当前的登录用户，并告知spring security框架
           List<Role> roleList = roleMapper.findRoleListByUserId(adminUser.getId());
           for (Role role : roleList) {
               List<Permission> permissionList = permissionMapper.findPermissionByRole(role.getId());
               authorityList.add(new SimpleGrantedAuthority("ROLE_"+role.getRoleKeyword()));
               for (Permission permission : permissionList) {
                   authorityList.add(new SimpleGrantedAuthority(permission.getPermissionKeyword()));
               }
           }
           UserDetails userDetails = new User(username,adminUser.getPassword(),authorityList);
           return userDetails;
       }
   }
   ~~~

4. 在接口上配置权限

   ~~~java
   @RestController
   @RequestMapping("/user")
   public class FunctionController {
       @GetMapping("findAll")
       @PreAuthorize("hasAuthority('USER_FINDALL')")
       public List<User> findAll(){
           return userService.findAll();
       }
   
       @GetMapping("findAge")
       @PreAuthorize("hasAuthority('USER_FINDAGE')")
       public List<User> findAge(){
           return userService.findAge();
       }
   
       @GetMapping("findById")
       @PreAuthorize("hasRole('ADMIN')")
       public User findById(@RequestParam("id") Long id){
           return userService.findById(id);
       }
   }
   ~~~

5. 在配置上开启权限认证

   ~~~java
   @Configuration
   @EnableGlobalMethodSecurity(prePostEnabled = true)
   public class SecurityConfig extends WebSecurityConfigurerAdapter {}
   ~~~

* 注意把原先config中的``.antMatchers("/user/findAll").hasRole("admin")``注掉

* 处理MyBatisPlus和MyBatis冲突的问题

  报错信息：

  ```
  org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)
  ```

  处理方法：

  1. 若pom同时存在mybatis-spring-boot-starter和mybatis-plus-boot-starter两个依赖，删除前者

  2. yaml新增配置

     ```yaml
     mybatis-plus:
       mapper-locations: classpath:com/example/mapper/*.xml
     ```

### 3.3 另一种方式进行授权

1. 修改配置

   ~~~java
     @Override
       protected void configure(HttpSecurity http) throws Exception {
   //        http.userDetailsService(securityUserService); //可不写
           http.authorizeRequests() //开启登录认证
   //                .antMatchers("/user/findAll").hasRole("admin")
                   .antMatchers("/login").permitAll()
               //使用访问控制 自己实现service处理，会接收两个参数 
                   .anyRequest().access("@authService.auth(request,authentication)")
   //                .anyRequest().authenticated() // 其他所有的请求 只需要登录即可
       }
   ~~~

2. 编写AuthService的auth方法

   ~~~java
   package com.example.service;
   
   @Service
   public class AuthService {
   
       @Autowired
       private AdminUserMapper adminUserMapper;
       @Autowired
       private RoleMapper roleMapper;
       @Autowired
       private PermissionMapper permissionMapper;
   
       public boolean auth(HttpServletRequest request, Authentication authentication){
           String requestURI = request.getRequestURI();
           Object principal = authentication.getPrincipal();
           if (principal == null || "anonymousUser".equals(principal)){
               //未登录
               return false;
           }
           UserDetails userDetails = (UserDetails) principal;
           Collection<? extends GrantedAuthority> authorities = userDetails.getAuthorities();
           for (GrantedAuthority authority : authorities) {
               MySimpleGrantedAuthority grantedAuthority = (MySimpleGrantedAuthority) authority;
               String[] paths = StringUtils.split(requestURI, "?");
               if (paths[0].equals(grantedAuthority.getPath())){
                   return true;
               }
           }
           return false;
       }
   }
   ~~~
   
3. 修改UserDetailService中 授权的实现，实现自定义的授权类，增加path属性

   ~~~java
   package com.example.security;
   
   import org.springframework.security.core.GrantedAuthority;
   
   public class MySimpleGrantedAuthority implements GrantedAuthority {
       private String authority;
       private String path;
   
       public MySimpleGrantedAuthority(){}
   
       public MySimpleGrantedAuthority(String authority){
           this.authority = authority;
       }
   
       public MySimpleGrantedAuthority(String authority,String path){
           this.authority = authority;
           this.path = path;
       }
   
       @Override
       public String getAuthority() {
           return authority;
       }
   
       public String getPath() {
           return path;
       }
   }
   ~~~


### 3.4 资源权限管理

1. web授权

   ```java
   @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests() //开启登录认证
                   .antMatchers("/user/findAll").hasRole("ADMIN") //访问接口需要的角色
                   .antMatchers("/user/getAll").hasAuthority("GETALL") //访问接口需要的权限
                   .antMatchers("/user/getSth").hasAnyAuthority("p1","p2")
                   .antMatchers("/login").permitAll()
   ```

   __注意__：规则的顺序是重要的，更具体的规则应该先写。先匹配到的规则会被执行。

2. 方法授权

   SpringSecurity2.0开始支持服务层方法的安全性支持。主要包括@PreAuthorize， @PostAuthorize，@Secured三类注解，首先在配置类上使用``@EnableGlobalMethodSecurity(prePostEnabled = true)``开启基于注解的安全性。

   ```java
   @Configuration
   @EnableGlobalMethodSecurity(securedEnabled = true)
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
   ```

   然后在方法（类或者接口上）添加注解就会限制对该方法的访问。SpringSecurity的原生注释支持为该方法定义了一组属性。这些属性将被传递给AccessDecisionManager以供它做出实际的决定。

   ```java
   public interface BookService{
       @Secured("IS_AUTHENTICATED_ANONYMOUSLY") //可以匿名访问
       public Account readAccount(Long id);
       
       @Secured("IS_AUTHENTICATED_ANONYMOUSLY") //可以匿名访问
       public Account[] findAccounts();
       
       @Secured("ROLE_TELLER") // 需要TELLER角色才可以访问
       public Account post(Account account, double amount);
   } 
   ```

   推荐使用@PreAuthorize和@PostAuthorize，首先开启该功能

   ```java
   @Configuration
   @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
   ```

   然后在方法上添加注解

   ```java
   @RestController
   @RequestMapping("/user")
   public class FunctionController {
       @GetMapping("/findAll")
       @PreAuthorize("hasAnyAuthority('p1', 'p2')")
       public String findAll(){
           return "GET ALL";
       }
   
       @GetMapping("/findById")
       @PreAuthorize("hasRole('ADMIN')")
       public String findById(){
           return "FIND BY ID";
       }
   
       @GetMapping("/findAge")
       @PreAuthorize("hasAuthority('USER_FINDAGE')")
       public String findAge(){
           return "FIND AGE";
       }
       
       @GetMapping("/func1")
       @PreAuthorize("hasAuthority('p1') and hasAuthority('p2')")
       public String func1(){
           return "FIND AGE";
       }
       
       @GetMapping("/func2")
       @PreAuthorize("isAnonymous()")
       public String func2(){
           return "FIND AGE";
       }
   }
   ```

# 四、会话管理

### 4.1 获取用户身份

```java
@RestController
@RequestMapping("/user")
public class FunctionController {
    @GetMapping("/getUserName")
    public String getUserName(){
        String userName = "";
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        Object principal = authentication.getPrincipal();
        if (principal instanceof UserDetails){
            UserDetails userDetails = (UserDetails) principal;
            userName = userDetails.getUsername();
        }else{
            userName = principal.toString();
        }
        return userName;
    }
}
```

### 4.2 会话Session控制

我们可以通过设置以下选项控制会话的创建和与Spring的交互方式

| 机制       | 描述                                                         |
| :--------- | ------------------------------------------------------------ |
| always     | 如果没有session存在就创建一个                                |
| ifRequired | 如果需要一个session就创建一个（默认）登陆时                  |
| never      | SpringSecurity将不会创建session，但是如果应用中其他地方创建了，那么SpringSecurity也会使用它 |
| stateless  | SpringSecurity将绝对不会创建session，也不使用session         |

通过以下配置方式对该选项进行配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception{
    http.sessionManagement()
        .sessionCreationPolicy(sessionCreationPolicy.IF_REQUIRED);
}
```

### 4.3安全会话cookie

可以使用httpOnly和secure标签来保护会话cookie

-  httpOnly：若为true，则浏览器脚本无法访问cookie
- secure：若为true，则cookie将仅通过https链接发送

springboot配置文件：

```properties
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
```

### 4.4 HttpSecurity常用方法及说明
HttpSecurity类中的方法较多，列举重要API如下。

| 方法                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| openidLogin()       | 用于基于 OpenId 的验证                                       |
| headers()           | 将安全标头添加到响应                                         |
| cors()              | 配置跨域资源共享（ CORS ）                                   |
| sessionManagement() | 允许配置会话管理                                             |
| portMapper()        | 允许配置一个PortMapper(HttpSecurity#(getSharedObject(class)))，其他提供SecurityConfigurer的对象使用。PortMapper 从 HTTP 重定向到 HTTPS 或者从 HTTPS 重定向到 HTTP。默认情况下，Spring Security使用一个PortMapperImpl映射 HTTP 端口8080到 HTTPS 端口8443，HTTP 端口80到 HTTPS 端口443 |
| jee()               | 配置基于容器的预认证。 在这种情况下，认证由Servlet容器管理   |
| x509()              | 配置基于x509的认证                                           |
| rememberMe          | 允许配置“记住我”的验证                                       |
| authorizeRequests() | 允许基于使用HttpServletRequest限制访问                       |
| requestCache()      | 允许配置请求缓存                                             |
| exceptionHandling() | 允许配置错误处理                                             |
| securityContext()   | 在HttpServletRequests之间的SecurityContextHolder上设置SecurityContext的管理。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |
| servletApi()        | 将HttpServletRequest方法与在其上找到的值集成到SecurityContext中。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |
| csrf()              | 添加 CSRF 支持，使用WebSecurityConfigurerAdapter时，默认启用 |
| logout()            | 添加退出登录支持。当使用WebSecurityConfigurerAdapter时，这将自动应用。默认情况是，访问URL”/ logout”，使HTTP Session无效来清除用户，清除已配置的任何#rememberMe()身份验证，清除SecurityContextHolder，然后重定向到”/login?success” |
| anonymous()         | 允许配置匿名用户的表示方法。 当与WebSecurityConfigurerAdapter结合使用时，这将自动应用。 默认情况下，匿名用户将使用org.springframework.security.authentication.AnonymousAuthenticationToken表示，并包含角色 “ROLE_ANONYMOUS” |
| formLogin()         | 指定支持基于表单的身份验证。如果未指定FormLoginConfigurer#loginPage(String)，则将生成默认登录页面 |
| oauth2Login()       | 根据外部OAuth 2.0或OpenID Connect 1.0提供程序配置身份验证    |
| requiresChannel()   | 配置通道安全。为了使该配置有用，必须提供至少一个到所需信道的映射 |
| httpBasic()         | 配置 Http Basic 验证                                         |
| addFilterAt()       | 在指定的Filter类的位置添加过滤器                             |

# 五、OAuth2.0简介

OAuth是一个开放标准，允许用户授权第三方应用访问，而不需要将用户名密码提供给第三方。Google，Yahoo，Microsoft都提供了OAuth认证服务。

OAuth大致流程：

![OAuth流程](https://images3.pianshen.com/511/82/82b327808e08f46d7d8469a25145d6c7.JPEG)

OAuth2.0包括以下角色：

1. 客户端

   本身不存储资源，需要通过资源拥有者的授权去请求资源服务器的资源，比如：Android客户端、浏览器端、微信客户端等。

2. 资源拥有者

   通常为用户，也可以是应用程序，即该资源的拥有者

3. 授权服务器

   用于服务提供商对资源拥有的身份进行认证，对访问资源进行授权，认证成功后会给客户端发放令牌(access_token)，作为客户端访问资源服务器的凭据。

4. 资源服务器

   存储资源的服务器。

服务提供商不会随便允许任意的客户端接入授权服务器。服务提供商会给准入的介入放一个身份，用于接入时的凭据：

__client_id__：客户端标识

__client_secret__：客户端密钥

### 5.1 Spring Cloud Security OAuth2.0

- Spring-Security-OAuth2.0是对OAuth2.0的一种实现，也可以与SpringCloud集成。OAuth2.0的服务提供方涵盖两个服务，即授权服务（Authentication Server亦即认证服务）和资源服务（Resource Server），使用Spring Security OAuth2.0可以选择将它们在一个服务中实现，也可以选择建立使用同一个授权服务的多个资源服务。
- 授权服务（Authentication Server）应包含对接入端以及登入用户的合法性进行验证并颁发token等功能，对令牌的请求端点由Spring MVC控制器进行实现，下面是配置一个认证服务器必须要实现的endpoints：
- AuthenticationEndPoint服务用于认证请求。默认URL:``/auth/authorize``

- TokenEndPoint服务用于令牌的请求。默认URL:``/oauth/token``

  资源服务（Resource Server）应包含对资源的保护功能，对非法请求进行拦截，对请求中token进行解析鉴权等，下面的过滤器用于实现OAuth2.0资源服务：

  - OAuth2AuthenticationProcessingFilter用来对请求给出的身份令牌解析鉴权