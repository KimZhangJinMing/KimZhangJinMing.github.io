### 1.SpringSecurity初体验

SpringBoot为SpringSecurity自动化配置了很多东西，只要把security的依赖导进来，所有的方法都会被security保护，需要登录才可以访问。

默认情况下，在启动SpringBoot项目的时候，会生成一个随机的密码，账户名是user，使用它可以进行登录。

如果使用postman进行访问，可以在Basis Auth中配置账号密码。如果验证成功，会自动跳转到/目录。

`此时可能发生的404错误是由于/目录下访问不到资源而导致的404.`

#### 1.1 配置用户名密码

如果配置了用户名和密码，SpringBoot项目启动的时候不再生成随机密码。

配置密码的方式有3种：

* 在配置文件中配置
* 在配置类中配置
* 保存到数据库



#### 1.2 在配置文件中配置

```yml
spring:
  security:
    user:
      name: user
      password: 123
      roles: # 角色，List<String>
        - admin
        - user
```

#### 1.3 在配置类中配置

需要继承WebSecurityConfigurerAdapter

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    
    // 密码不加密,可以使用明文密码，已经过时
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 配置多个用户的账号密码角色，其中密码能配置成明文是因为NoOpPasswordEncoder的作用
        auth.inMemoryAuthentication()
                .withUser("user")
                .roles("admin")
                .password("123")
                .and()
                .withUser("ming")
                .roles("user")
                .password("123");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        		// 开启登录配置
                http.authorizeRequests()
            	// /admin/**的url需要admin角色才能访问
                .antMatchers("/admin/**")
                .hasRole("admin")
            	// /user/**的url,admin和user角色都可以访问
                .antMatchers("/user/**")
                .hasAnyRole("admin","user")
            	// 其他的请求，只要登录了就可以访问
                .anyRequest()
                .authenticated()
                .and()
            	// 登录的时候访问的url修改为/doLogin,账号和密码的参数名称是user和pwd
                .formLogin()
                .loginProcessingUrl("/doLogin")
                .usernameParameter("user")
                .passwordParameter("pwd")
            	// 登录成功后的处理，这里返回JSON格式的数据
                .successHandler( (request,response,authentication) -> {
                    response.setContentType("application/json;charset=utf-8");
                    PrintWriter writer = response.getWriter();
                    Map<String,Object> map = new HashMap<>();
                    map.put("status",200);
                    map.put("message",authentication.getPrincipal());
                    String result = new ObjectMapper().writeValueAsString(map);
                    writer.flush();
                    writer.write(result);
                })
                // 登录失败后的处理，这里返回JSON格式的数据
                .failureHandler( (request,response,exception) -> {
                    response.setContentType("application/json;charset=utf-8");
                    PrintWriter writer = response.getWriter();
                    Map<String,Object> map = new HashMap<>();
                    map.put("status",500);
                    map.put("message",exception.getMessage());
                    String result = new ObjectMapper().writeValueAsString(map);
                    writer.flush();
                    writer.write(result);
                })
            	// 和表单登录相关的接口都可以访问
                .permitAll()
                .and()
            	// 退出登录时访问的url
                .logout()
                .logoutUrl("/logout")
            	// 成功退出时的处理，返回JSON数据
                .logoutSuccessHandler( (request,response,authentication) -> {
                    response.setContentType("application/json;charset=utf-8");
                    PrintWriter writer = response.getWriter();
                    Map<String,Object> map = new HashMap<>();
                    map.put("status",200);
                    map.put("message","注销成功");
                    String result = new ObjectMapper().writeValueAsString(map);
                    writer.flush();
                    writer.write(result);
                })
                .and()
            	// 禁用CSRF
                .csrf()
                .disable();
    }
}

```

由于 Spring Security 支持多种数据源，例如内存、数据库、LDAP 等，这些不同来源的数据被共同封装成了一个 UserDetailService 接口，任何实现了该接口的对象都可以作为认证数据源。

因此我们还可以通过重写 WebSecurityConfigurerAdapter 中的 userDetailsService 方法来提供一个 UserDetailService 实例进而配置多个用户：

```java
@Override
@Bean
protected UserDetailsService userDetailsService() {
    // JdbcUserDetailsManager、InMemoryUserDetailsManager都是UserDetailService的实例
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    // 参数是一个UserDetails,通过User.withUsername等方法返回的UserBuilder.build返回一个UserDetails实例
    manager.createUser(User.withUsername("ming").password("$2a$10$vcMANJ1FiElZyS31BwuiwusGihNfcoGGQJJtdJEqi7C4Ukpv26fze").roles("user").build());
    return manager;
}
```

`但是需要注意的是，两种方式只能选择其中一种。`

#### 1.4 在数据库中配置

从数据库中配置就需要创建数据表保存账户、角色信息。

`建表语句：`

```sql
/*
SQLyog Ultimate v12.4.3 (64 bit)
MySQL - 5.7.17-log : Database - security
*********************************************************************
*/

/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`security` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `security`;

/*Table structure for table `role` */

DROP TABLE IF EXISTS `role`;

CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `nameZh` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Data for the table `role` */

insert  into `role`(`id`,`name`,`nameZh`) values 
(1,'ROLE_dba','数据库管理员'),
(2,'ROLE_admin','系统管理员'),
(3,'ROLE_user','用户');

/*Table structure for table `user` */

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `enabled` tinyint(1) DEFAULT NULL,
  `locked` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Data for the table `user` */

insert  into `user`(`id`,`username`,`password`,`enabled`,`locked`) values 
(1,'root','$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq',1,0),
(2,'admin','$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq',1,0),
(3,'sang','$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq',1,0);

/*Table structure for table `user_role` */

DROP TABLE IF EXISTS `user_role`;

CREATE TABLE `user_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `uid` int(11) DEFAULT NULL,
  `rid` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;

/*Data for the table `user_role` */

insert  into `user_role`(`id`,`uid`,`rid`) values 
(1,1,1),
(2,1,2),
(3,2,2),
(4,3,3);

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

> user

| id   | username | password                                                     | enabled | locker |
| ---- | -------- | ------------------------------------------------------------ | ------- | ------ |
| 1    | root     | $2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq | 1       | 0      |
| 2    | admin    | $2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq | 1       | 0      |
| 3    | sang     | $2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq | 1       | 0      |

> role

| id   | name       | nameZh       |
| ---- | ---------- | ------------ |
| 1    | ROLE_dba   | 数据库管理员 |
| 2    | ROLE_admin | 系统管理员   |
| 3    | ROLE_user  | 用户         |

> user_role

root用户拥有dba,admin角色。

admin用户拥有admin角色。

user用户拥有user角色。

| id   | uid  | rid  |
| ---- | ---- | ---- |
| 1    | 1    | 1    |
| 2    | 1    | 2    |
| 3    | 2    | 2    |
| 4    | 3    | 3    |

创建好表后，我们需要连接数据库，这里我就使用JPA来连接数据库，在连接数据库之前记得导入mysql、druid、jpa的依赖。

我们还需编写配置文件和创建实体User,Role。

`配置文件:`

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/security?useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: 123
  jpa:
    hibernate:
      ddl-auto: none
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    database: mysql
    database-platform: mysql
    show-sql: true
    properties:
      hibernate:
        # format-sql必须放在properties.hibernate才有效,而且使用的是下划线
        format_sql: true
        dialect: org.hibernate.dialect.MySQL57Dialect
```

User类实现了`UserDetails`接口，并实现了其方法。UserDetails接口是告诉Spring Security，我这个User类的哪些属性对应着登录名，密码等。

我们希望在验证用户是否在系统中存在的时候，把该用户的角色给它赋值上。这里使用了JPA的导航属性，在查询出User类的时候，就会把对应的角色给赋值了。

```java
@Setter
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    @Column(name = "username")
    private String userName;
    private String password;
    private Boolean enabled;
    private Boolean locked;
    // 这里设置roles属性不能使用懒加载，懒加载会报错
    // 在查询出roles属性的时候才使用懒加载
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "user_role",joinColumns = @JoinColumn(name = "uid"),
               inverseJoinColumns = @JoinColumn(name = "rid"))
    private List<Role> roles;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.roles.stream()
                .map(r -> new SimpleGrantedAuthority(r.getName()))
                .collect(Collectors.toList());
    }

    @Override
    public String getUsername() {
        return this.userName;
    }

    /**
     * 账号是否还没过期
     * @return
     */
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 账号是否未被锁定
     * @return
     */
    @Override
    public boolean isAccountNonLocked() {
        return !locked;
    }

    /**
     * 凭证是否未过期
     * @return
     */
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 账号是否可用
     * @return
     */
    @Override
    public boolean isEnabled() {
        return enabled;
    }
}

```

Role:

```java
@Setter
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    // 这里的column实际上是无效的，因为@Column指定的name属性和实体属性名称相同就会失效。
    // 需要到配置文件中指定JPA的命名策略
    @Column(name = "nameZh")
    private String nameZh;
}
```

service实现了UserDetailsService,稍后在配置Spring Security的用户时会用到。

```java
@Service
public class UserService implements UserDetailsService {

    @Autowired
    private UserDao userDao;

    /**
     * 参数username就是前端参数登录时传递过来的
     * 只需要判断登录的用户在系统存在与否
     * 至于密码校验那些，交给Spring Security
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user =  userDao.findUserByUserName(username)
                .orElseThrow(() -> new UsernameNotFoundException("username not found"));
        // 设置角色，使用JPA的导航属性
        return user;
    }
}
```

`UserDao:`

```java
public interface UserDao extends JpaRepository<User,Integer> {

    Optional<User> findUserByUserName(String username);
}
```

`DataBaseSecurityConfiguration:`

配置Spring Security的用户信息，httpConfig，密码编码等。

```java
@Configuration
public class DataBaseSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/dba/**")
                .hasRole("dba")
                .antMatchers("/admin/**")
                .hasRole("admin")
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/doLogin")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}
```

`HelloController:`

编写3个接口，分别测试dba,admin,user的访问权限。

```java
@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;
    
    @GetMapping("/dba/hello")
    public String dba() {
        return "hello,dba";
    }

    @GetMapping("/admin/hello")
    public String admin() {
        return "hello,admin";
    }

    @GetMapping("/user/hello")
    public String user() {
        return "hello,user";
    }
}

```

> 小结：使用数据库配置Spring Security的账户密码
>
> * 实体类需要实现UserDetail接口，提供给Spring Security一些信息做为验证的依据
> * Spring Security在配置用户信息的时候需要一个UserDetailService，我们需要创建一个Service实现UserDetailService接口，查询前端登录的用户在系统中是否存在，其他验证工作交给Spring Security即可

> 使用JDBC从数据库中配置

在org\springframework\security\spring-security-core\5.4.2\spring-security-core-5.4.2.jar!\org\springframework\security\core\userdetails\jdbc\users.ddl

如果是使用Mysql数据库，把varchar_ignorecase改成varchar就可以了

```sql
create table users(username varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not null,enabled boolean not null);
create table authorities (username varchar_ignorecase(50) not null,authority varchar_ignorecase(50) not null,constraint fk_authorities_users foreign key(username) references users(username));
create unique index ix_auth_username on authorities (username,authority);
```

使用JdbcUserDetailsManager

```java
@Autowired
DataSource dataSource;

@Override
@Bean
protected UserDetailsService userDetailsService() {
    // 需要datasource
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
    // 系统每次启动都会执行这段代码，需要判断用户在数据库中是否存在，避免多次创建用户
    if(!manager.userExists("user")){
        manager.createUser(User.withUsername("user").password("$2a$10$vcMANJ1FiElZyS31BwuiwusGihNfcoGGQJJtdJEqi7C4Ukpv26fze").roles("admin").build());
    }
    if(!manager.userExists("ming")) {
        manager.createUser(User.withUsername("ming").password("$2a$10$vcMANJ1FiElZyS31BwuiwusGihNfcoGGQJJtdJEqi7C4Ukpv26fze").roles("user").build());
    }
    return manager;
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService());
}
```





#### 1.5 多个HttpConfig的配置

HttpConfig是配置访问规则的，可以存在多个。

如果需要多个配置多个HttpConfig，可以使用多个静态内部类继承WebSecurityConfigurerAdapter来注册多个。

多个类需要使用@Order来指定执行顺序。数值越小，优先级越高。

```java
@Configuration
public class MultiSecurityConfiguration {

    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
	
    // 即使多个HttpConfig,账户密码也是一样的
    @Autowired
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("user")
                .roles("admin")
                .password("111")
                .and()
                .withUser("ming")
                .roles("user")
                .password("222");
    }

    @Configuration
    @Order(1)
    public static class AdminSecurityConfiguration extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            // /admin/**下的所有请求都需要admin角色才能访问
            http.antMatcher("/admin/**")
                    .authorizeRequests()
                    .anyRequest()
                    .hasRole("admin");
        }
    }

    @Configuration
    public static class OtherSecurityConfiguration extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/user/**")
                    .hasAnyRole("admin","user")
                    .anyRequest()
                    .authenticated()
                    .and()
                    .formLogin()
                    .loginProcessingUrl("/doLogin")
                    .permitAll()
                    .and()
                    .csrf()
                    .disable();
        }
    }
}

```



#### 1.6 加密密码

Spring Security提供了BCryptPasswordEncoder类，可以加密密码。

即使明文是一致的，生成的加密密码也会不一样，先来看一个简单的测试：

```java
@SpringBootTest
public class EncoderTest {

    @Test
    public void test(){
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        String password1 = encoder.encode("123");
        String password2 = encoder.encode("123");
        System.out.println(password1);
        System.out.println(password2);
    }
}
```

明文密码123生成的加密密码是不一样的。

```txt
$2a$10$CuU3nXzZ.DtVmI5hxxNS1exrV0nSECqL182g/ockvp8tylmdYb67O
$2a$10$Or.xAznkldCAQKnuFyKMG.V9.VAfI0pk7tUeVdNGP9vZDFT5stj72
```

我们可以在之前的配置类中使用加密密码：

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(); // 使用BCryptPasswordEncoder
}

// 将明文密码123通过BCryptPasswordEncoder生成的加密密码配置在这里，通过密码123就能登录
@Autowired
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user")
        .roles("admin")
        .password("$2a$10$CuU3nXzZ.DtVmI5hxxNS1exrV0nSECqL182g/ockvp8tylmdYb67O")
        .and()
        .withUser("ming")
        .roles("user")
        .password("$2a$10$Or.xAznkldCAQKnuFyKMG.V9.VAfI0pk7tUeVdNGP9vZDFT5stj72");
}
```



#### 1.7 方法级别的权限

可以使用@PreAuthorize、@PostAuthorize、@Secured三个注解对方法的访问进行限制。

同时需要配置@EnableGlobalMethodSecurity注解开启全局方法限制。

controller中需要访问service层的方法，@EnableGlobalMethodSecurity加在controller层。

```java
@RestController
// prePostEnabled：开启@PreAuthorize、@PostAuthorize的权限验证
// securedEnabled：开启@Secured的权限验证
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class HelloController {

    @Autowired
    private HelloService helloService;

    @GetMapping("/hello1")
    public String hello1() {
        return helloService.hello1();
    }
    @GetMapping("/hello2")
    public String hello2() {
        return helloService.hello2();
    }
    @GetMapping("/hello3")
    public String hello3() {
       return helloService.hello3();
    }
}

```

service层中的方法中使用@PreAuthorize、@PostAuthorize、@Secured三个注解对方法的访问进行限制。

@PreAuthorize、@PostAuthorize可以使用表达式。

@Secured只能使用ROLE_角色名称，并且需要注意大小写。如果大小写写错，这个方法限制没生效，所有的用户都可以访问该方法。

```java
@Service
public class HelloService {
	
    // admin、user角色都可以访问该方法
    @PreAuthorize("hasAnyRole('admin','user')")
    public String hello1() {
        return "hello1";
    }
	// 只有user角色能访问该方法
    @Secured("ROLE_user")
    public String hello2() {
        return "hello2";
    }
	// 只有admin角色能访问该方法
    @PreAuthorize("hasRole('admin')")
    public String hello3() {
        return "hello3";
    }
}
```



#### 1.8 角色继承

上级可能具备下级的所有权限，如果使用角色继承，这个功能就很好实现，我们只需要在 SecurityConfig 中添加如下代码来配置角色继承关系即可。

角色继承的配置只需要在配置类中，将RoleHierarchy加入到容器即可。

`注意，在配置时，需要给角色手动加上 `ROLE_` 前缀。下面的配置表示 `ROLE_admin` 自动具备 `ROLE_user` 的权限。`.

如果有多个角色继承关系的配置，使用空格分割开。比如`ROLE_admin > ROLE_user  ROLE_root > ROLE_user`

但是SpringBoot 2.4.1的版本是使用\n分隔符,`ROLE_admin > ROLE_user \n ROLE_root > ROLE_user`

```java
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_admin > ROLE_user");
    return roleHierarchy;
}
```



#### 1.9 动态权限配置

前面我们配置的HttpConfig都是写在配置类中的，在真实的项目中，这显然不合理。

真实项目中，应该从把哪些角色可以访问哪些资源的信息保存到数据库中。修改了数据库的信息，就相当于动态修改了权限。

在之前的security库中添加menu和menu_role两张表。

menu代表着可以访问的资源的路径，menu_role代表着访问资源的路径和角色之间的关系。

```sql
/*Table structure for table `menu` */

DROP TABLE IF EXISTS `menu`;

CREATE TABLE `menu` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `pattern` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Data for the table `menu` */

insert  into `menu`(`id`,`pattern`) values 
(1,'/db/**'),
(2,'/admin/**'),
(3,'/user/**');

/*Table structure for table `menu_role` */

DROP TABLE IF EXISTS `menu_role`;

CREATE TABLE `menu_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `mid` int(11) DEFAULT NULL,
  `rid` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Data for the table `menu_role` */

insert  into `menu_role`(`id`,`mid`,`rid`) values 
(1,1,1),
(2,2,2),
(3,3,3);
```

> menu

| id   | pattern   |
| ---- | --------- |
| 1    | /dba/**   |
| 2    | /admin/** |
| 3    | /user/**  |

> menu_role

| id   | mid  | rid  |
| ---- | ---- | ---- |
| 1    | 1    | 1    |
| 2    | 2    | 2    |
| 3    | 3    | 3    |

新增实体类`Menu:`

一个资源可能有多个角色可以访问，有List<Role>属性代表可以访问该资源的所有角色。

```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
public class Menu {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String pattern;
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "menu_role",joinColumns = @JoinColumn(name = "mid"),
            inverseJoinColumns = @JoinColumn(name = "rid"))
    private List<Role> roles;
}

```

动态配置权限需要两个类：

* 根据当前的请求，从数据库中查询出哪些角色可以访问该资源

* 根据数据库中查询出来的角色和当前登录用户的角色进行对比,校验能否访问

`RequestUrlFilter:`

```java
/**
 *@Author Ming
 *@Date 2021/01/16 15:39
 *@Description 根据当前的请求路径,从数据库中获取访问当前路径需要的角色列表
 */
@Component
public class RequestUrlFilter implements FilterInvocationSecurityMetadataSource {

    @Autowired
    private MenuDao menuDao;
    // 注意是new出来的,不是注入
    private AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        FilterInvocation filterInvocation = (FilterInvocation) o;
        // 获取当前请求路径
        String requestUrl = filterInvocation.getRequestUrl();
        // 获取数据库中的路径
        List<Menu> allMenu = menuDao.findAll();
        // 获取当前请求路径需要的角色列表
        List<String> roleNames = allMenu.stream()
                .filter(menu -> antPathMatcher.match(menu.getPattern(), requestUrl))
                .flatMap(menu -> menu.getRoles().stream())
                .map(r -> r.getName())
                .collect(Collectors.toList());
        // list转数组,如果访问的路径与数据库中的路径不匹配，返回一个标识ROLE_login(自定义的,后面判断的时候用到)
        String[] roleArray = new String[roleNames.size()];
        // 不能返回空集合，否则进不了DecisionManager
        return SecurityConfig.createList(CollectionUtils.isEmpty(roleNames) ? new String[]{"ROLE_login"}
                : roleNames.toArray(roleArray));
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}

```

> Lambda写法

```java
/**
 *@Author Ming
 *@Date 2021/01/22 22:44
 *@Description 返回有权访问当前请求的角色列表,
 * 如果当前请求在数据库中找不到对应的角色列表,不能返回空集合,而是返回一个标识,
 * 在DecisionManager中可以根据这个标识判断
 *
 */
@Component
public class MenuRoleFilter implements FilterInvocationSecurityMetadataSource {

    @Autowired
    private MenuMapper menuMapper;
    private AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        String requestUrl = ((FilterInvocation) o).getRequestUrl();
        // 获取所有菜单和菜单对应的角色
        List<Menu> menuList = menuMapper.getAllMenuWithRole();
        // 访问当前请求需要的角色列表,如果匹配不到请求,返回空集合
        List<String> roleNameList = menuList.stream()
                .filter(m -> m.getPath()!=null)
                .filter(m -> antPathMatcher.match(m.getPath(), requestUrl))
                .flatMap(m -> m.getRoles().stream())
                .map(Role::getName)
                .collect(Collectors.toList());
        String[] roleNameArray = new String[roleNameList.size()];
        // 不能返回空集合，否则进不了DecisionManager
        return roleNameList.size() == 0 ?SecurityConfig.createList("ROLE_LOGIN")
                : SecurityConfig.createList(roleNameArray);
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}

```



`MyAccessDecisionManager:`

```java
/**
 *@Author Ming
 *@Date 2021/01/16 15:59
 *@Description 访问当前路径所需要的角色和数据库中的角色进行校验


 */
@Component
public class MyAccessDecisionManager implements AccessDecisionManager {
    /**
     * 决定当前登录的用户是否有权限访问当前访问的路径
     * @param authentication 用户的登录信息
     * @param o FilterInvocation对象
     * @param collection FilterInvocationSecurityMetadataSource返回的角色列表
     * @throws AccessDeniedException 没有权限访问时抛出的异常
     * @throws InsufficientAuthenticationException 有部分权限，但是权限不足以访问的异常
     */
    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection) throws AccessDeniedException, InsufficientAuthenticationException {
        for (ConfigAttribute configAttribute : collection) {
            String needRole = configAttribute.getAttribute();
            if ("ROLE_LOGIN".equals(needRole)) {
                // 当前访问的请求在数据库中匹配不到相应的请求
                if (authentication instanceof AnonymousAuthenticationToken) {
                    throw new InsufficientAuthenticationException("请先登录");
                } else {
                    return;

                }
            } else {
                // 获取当前用户的角色列表
                Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
                for (GrantedAuthority authority : authorities) {
                    if(needRole.equals(authority.getAuthority())){
                        return;
                    }
                }
            }

        }
        throw new InsufficientAuthenticationException("权限不足,请联系管理员..");
    }


    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}

```

在配置类中注入`RequestUrlFilter,MyAccessDecisionManager`,并在config中配置它们，完整的配置类如下：

`如果使用动态控制权限，就不能再配置anyRequest().antherticated()`

```java
@Configuration
public class DataBaseSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;
    @Autowired
    private RequestUrlFilter requestUrlFilter;
    @Autowired
    private MyAccessDecisionManager myAccessDecisionManager;

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }

    @Bean
    RoleHierarchy roleHierarchy() {
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        roleHierarchy.setHierarchy("ROLE_root > ROLE_user \n ROLE_admin > ROLE_user");
        return roleHierarchy;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O o) {
                        o.setSecurityMetadataSource(requestUrlFilter);
                        o.setAccessDecisionManager(myAccessDecisionManager);
                        return o;
                    }
                })
                .and()
                .formLogin()
                .loginProcessingUrl("/doLogin")
                .permitAll()
                .and()
                .csrf()
                .disable();
    }
}

```



#### 2.0 从内存中获取登录信息

```java
Hr hr = (Hr) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
```



#### 2.1 JSON格式登陆

Spring Security默认是使用Key-Value的形式进行登陆的。也就是默认的/login请求只支持Key-Value的POST请求。

用户登陆的用户名/密码是在`UsernamePasswordAuthenticationFilter`类中处理的。

```java
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        } else {
            String username = this.obtainUsername(request);
            username = username != null ? username : "";
            username = username.trim();
            String password = this.obtainPassword(request);
            password = password != null ? password : "";
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
            this.setDetails(request, authRequest);
            return this.getAuthenticationManager().authenticate(authRequest);
        }
    }
```

从这段代码中，我们就可以看出来为什么 Spring Security 默认是通过 key/value 的形式来传递登录参数，因为它处理的方式就是 request.getParameter。

所以我们要定义成 JSON 的，思路很简单，就是自定义来定义一个过滤器代替 `UsernamePasswordAuthenticationFilter` ，重写attemptAuthentication方法.

`JsonAuthenticationFilter:` 定义一个过滤器继承`UsernamePasswordAuthenticationFilter`.

```java
public class JsonAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        // 只支持POST方法
        if(!"POST".equals(request.getMethod())) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        // 处理JSON格式的数据,从reqeust中用流的方式读取成Map
        if (MediaType.APPLICATION_JSON_VALUE.equals(request.getContentType())) {
            try {
                Map<String, String> params = new ObjectMapper().readValue(request.getInputStream(), Map.class);
                String username = params.get(this.getUsernameParameter());
                username = username != null ? username.trim() : "";
                String password = params.get(this.getPasswordParameter());
                password = password != null ? password.trim() : "";
                UsernamePasswordAuthenticationToken authRequest =
                        new UsernamePasswordAuthenticationToken(username, password);
                this.setDetails(request,authRequest);
                return this.getAuthenticationManager().authenticate(authRequest);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        // 这里是调用父类处理Key-Value的方法,如果使用this,就造成递归调用了
        return super.attemptAuthentication(request,response);
    }
}
```

接下来要在配置类中创建自己的Filter类，并配置到HttpConfig中,filter中能设置的属性跟formLogin的差不多。Filter中设置的和formLogin设置的都能起效，但一般只需要设置一个即可，完整的配置类如下:

`SecurityConfiguration`

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    protected PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       auth.inMemoryAuthentication()
               .withUser("user")
               .password("$2a$10$vcMANJ1FiElZyS31BwuiwusGihNfcoGGQJJtdJEqi7C4Ukpv26fze")
               .roles("user","admin");
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().mvcMatchers("/js/**","/css/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
            	// 注册自己编写的Filter
                .addFilterAt(jsonAuthenticationFilter(), JsonAuthenticationFilter.class)
            	// 这里设置formLogin与Filter的不冲突
                .formLogin()
                .permitAll()
                .and()
                .csrf()
                .disable();
    }

    @Bean
    protected JsonAuthenticationFilter jsonAuthenticationFilter() throws Exception {
        JsonAuthenticationFilter filter = new JsonAuthenticationFilter();
        filter.setUsernameParameter("username");
        filter.setPasswordParameter("password");
        // 这个非常重要
        filter.setAuthenticationManager(authenticationManagerBean());
        // 类似于loginProcessUrl
        filter.setFilterProcessesUrl("/doLogin");
        filter.setAuthenticationSuccessHandler((request,response,auth) -> {
            response.setContentType("application/json;charset=utf-8");
            PrintWriter writer = response.getWriter();
            Map<String,Object> result = new HashMap<>();
            result.put("status",200);
            result.put("data",auth.getPrincipal());
            writer.write(new ObjectMapper().writeValueAsString(result));
            writer.flush();
            writer.close();
        });
        filter.setAuthenticationFailureHandler( (request,response,e) -> {
            response.setContentType("application/json;charset=utf-8");
            PrintWriter writer = response.getWriter();
            Map<String,Object> result = new HashMap<>();
            result.put("status",500);
            result.put("data",e.getMessage());
            writer.write(new ObjectMapper().writeValueAsString(result));
            writer.flush();
            writer.close();
        });

        return filter;
    }

}

```



#### 2.2 持久化令牌

当每次使用用户名/密码登陆的时候，会生成一个series和token，浏览器把它保存在cookie中。当退出浏览器再重新访问的时候，还是同一个cookie，并不会重新生成新的token。

而当在别端访问的时候，使用用户名/密码登陆的时候，会生成新的token，之前的浏览器cookie就会失效，从而实现多端踢下线的功能。

如果之前在别端访问过，又没有退出，是可以继续访问的。

如果使用Spring Security自带的JdbcPersistentTokenRepository,需要添加一张数据表，JdbcPersistentTokenRepository会操作这张表。

```sql
CREATE TABLE `persistent_logins` (
  `username` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `series` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `token` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
  `last_used` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

配置类中:

```java
@Bean
PersistentTokenRepository persistentTokenRepository() {
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    return jdbcTokenRepository;
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/**")
        .hasRole("admin")
        .antMatchers("/user/**")
        .hasRole("user")
        .anyRequest()
        .authenticated()
        .and()
        .addFilterAt(jsonAuthenticationFilter(), JsonAuthenticationFilter.class)
        .formLogin()
        .permitAll()
        .and()
        .rememberMe()
        .key("ming")
        .tokenRepository(persistentTokenRepository())
        .and()
        .csrf()
        .disable();
}
```



#### 2.2 自定义验证逻辑-验证码

首先，需要引入验证码的框架:

```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

配置验证码,这里主要是设置一下验证码图片的宽高等。

```java
@Configuration
public class KaptchaConfiguration {

    @Bean
    Producer producer() {
        Properties properties = new Properties();
        properties.setProperty("kaptcha.image.width","150");
        properties.setProperty("kaptcha.image.height","50");
        properties.setProperty("kaptcha.textproducer.char.string","0123456789");
        properties.setProperty("kaptcha.textproducer.char.length","4");
        Config config = new Config(properties);
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}
```

接口返回验证码图片:

```java
@Controller
public class KaptchaController {

    @Autowired
    Producer producer;

    @GetMapping("code.jpg")
    public void codeImage(HttpServletRequest request, HttpServletResponse response) {
        response.setContentType(MediaType.IMAGE_JPEG_VALUE);
        String text = producer.createText();
        BufferedImage image = producer.createImage(text);
        request.getSession().setAttribute("vCode",text);
        try {
            ServletOutputStream out = response.getOutputStream();
            ImageIO.write(image,"jpg",out);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

实现自己的AutherationProvider,继承自DaoAuthenticationProvider，因为所有的账户/密码登陆的方式都会经过这里，我们可以在这里添加验证码的逻辑。

```java
public class MyAuthenticationProvider extends DaoAuthenticationProvider {
    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        // 验证码逻辑
        HttpServletRequest request =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String code = request.getParameter("code");
        String vCode = (String) request.getSession().getAttribute("vCode");
        if(code == null || vCode == null || !code.equals(vCode)) {
            throw new BadCredentialsException("验证码错误");
        }
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
```

配置类中添加自己写的AuthenticationProvider，同时放开访问验证码图片的接口：

```java
@Bean
@Override
protected AuthenticationManager authenticationManager() {
    return new ProviderManager(myAuthenticationProvider());
}

@Bean
MyAuthenticationProvider myAuthenticationProvider() {
    MyAuthenticationProvider provider = new MyAuthenticationProvider();
    provider.setPasswordEncoder(passwordEncoder());
    provider.setUserDetailsService(userDetailsService());
    return provider;
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/code.jpg")
        .permitAll()
        .antMatchers("/admin/**")
        .hasRole("admin")
        .antMatchers("/user/**")
        .hasRole("user")
        .anyRequest()
        .authenticated()
        .and()
        .addFilterAt(jsonAuthenticationFilter(), JsonAuthenticationFilter.class)
        .formLogin()
        .permitAll()
        .and()
        .rememberMe()
        // .key("ming")
        .tokenRepository(persistentTokenRepository())
        .and()
        .csrf()
        .disable();
}
```



#### 2.3 异常处理

当未登录直接访问url时，拦截器(DecisionManager)会抛出异常,并重定向到/login登录页面，此时前端是没有通过代理的，直接访问url，就会出现跨域的问题。

有一种解决方案是在/login上添加跨域的注解,但这不是很好，我希望后端返回`未登录`的信息给前端，这就需要异常处理了。

```java

```

