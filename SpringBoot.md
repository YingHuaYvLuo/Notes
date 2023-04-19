# 请求

## 映射

@RequestMapping注解是一个用来处理请求地址映射的注解，可用于映射一个请求或一个方法，可以用在类或方法上。

### 标注在方法上

用于方法上，表示在类的父路径下追加方法上注解中的地址将会访问到该方法。

```java
@RequestMapping("/testRequest")
public String testRequest(){
    return "success";
}
```

### 标注在类和方法上

用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

```java
@Controller
@RequestMapping(“/hello”)
public class RequestMappingController {
    @RequestMapping("/testRequest")
	public String testRequest(){
    	return "success";
	}
}
```

## 设置请求参数默认值

@RequestParam(defaultValue="1") 

```java
```

## 简单参数

**~~原始方式~~**

```java
@RequestMapping("/simpleParam")
public String simpleParam(HttpServletRequest request){
    String name = request.getParameter("name");
    String ageStr = request.getParameter("age");
    int age = Integer.parseInt(ageStr);
    System.out.println(name+"  :  "+age);
    return "OK";
}
```

**SpringBoot方式**

参数名与形参变量名相同，定义形参即可接收参数

```java
@RequestMapping("/simpleParam")
public String simpleParam(String name , Integer age){
    System.out.println(name+"  :  "+age);
    return "OK";
}
```

如果方法形参名称与请求参数名称不匹配，可以使用 `@RequestParam` 完成映射

```java
@RequestMapping("/simpleParam")
public String simpleParam(@RequestParam(name = "name") String username , Integer age){
    System.out.println(username + " : " + age);
    return "OK";
}

//@RequestParam中的required属性默认为true，代表该请求参数必须传递，如果不传递将报错。 如果该参数是可选的，可以将required属性设置为false。
```

## 实体参数

请求参数名与形参对象属性名相同，定义POJO接收即可

```java
@RequestMapping("/simplePojo")
public String simplePojo(User user){
    System.out.println(user);
    return "OK";
}

public class User {
    private String name;
    private Integer age;
}
```

请求参数名与形参对象属性名相同，按照对象层次结构关系即可接收嵌套POJO属性参数。

```java
@RequestMapping("/complexPojo")
public String complexPojo(User user){
    System.out.println(user);
    return "OK";
}

public class User {
    private String name;
    private Integer age;
    private Address address;
}

public class Address {
    private String province;
    private String city;
}
```

## 数组集合参数

数组参数：请求参数名与形参数组名称相同且请求参数为多个，定义数组类型形参即可接收参数

```java
@RequestMapping("/arrayParam")
public String arrayParam(String[] hobby){
    System.out.println(Arrays.toString(hobby));
    return "OK";
}
```

集合参数：请求参数名与形参集合名称相同且请求参数为多个，`@RequestParam` 绑定参数关系

```java
@RequestMapping("/listParam")
public String listParam(@RequestParam List<String> hobby){
    System.out.println(hobby);
    return "OK";
}
```

## 日期参数

使用 @DateTimeFormat 注解完成日期参数格式转换

```java
@RequestMapping("/dateParam")
public String dateParam(@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime updateTime){
    System.out.println(updateTime);
    return "OK";
}
```

## JSON参数

JSON数据键名与形参对象属性名相同，定义POJO类型形参即可接收参数，需要使用 @RequestBody 标识

```java
@RequestMapping("/jsonParam")
public String jsonParam(@RequestBody User user){
    System.out.println(user);
    return "OK";
}

public class User {
    private String name;
    private Integer age;
    private Address address;
}

public class Address {
    private String province;
    private String city;
}
```

## 路径参数

通过请求URL直接传递参数，使用{…}来标识该路径参数，需要使用 @PathVariable 获取路径参数

```java
@RequestMapping("/path/{id}")
public String pathParam(@PathVariable Integer id){
    System.out.println(id);
    return "OK";
}

@RequestMapping("/path/{id}/{name}")
public String pathParam2(@PathVariable Integer id, @PathVariable String name){
    System.out.println(id+ " : " +name);
    return "OK";
}
```

## 文件参数

### 前端页面三要素

- 表单项 type=“file”
- 表单提交方式 post
- 表单的enctype属性 multipart/form-data

```html
<form action="/upload" method="post" enctype="multipart/form-data">
    姓名: <input type="text" name="username"><br>
    年龄: <input type="text" name="age"><br>
    头像: <input type="file" name="image"><br>
    <input type="submit" value="提交">
</form>
```

### 本地存储

- 使用MultipartFile接收
  - String getOriginalFilename(); //获取原始文件名
  - void transferTo(File dest); //将接收的文件转存到磁盘文件中
  - long getSize(); //获取文件的大小，单位：字节
  - byte[] getBytes(); //获取文件内容的字节数组
  - InputStream getInputStream(); //获取接收到的文件内容的输入流

```java
@RestController
public class UploadController {
    @PostMapping("/upload")
    public Result upload(MultipartFile image) throws IOException {
        //获取原始文件名
        String originalFilename = image.getOriginalFilename();
        //构建新的文件名
        String newFileName = UUID.randomUUID().toString()+originalFilename.substring(originalFilename.lastIndexOf("."));
        //将文件保存在服务器端 E:/images/ 目录下
        image.transferTo(new File("E:/images/"+newFileName));
        return Result.success();
    }
}
```

### 相关配置

```properties
#配置单个文件最大上传大小
spring.servlet.multipart.max-file-size=10MB
#配置单个请求最大上传大小(一次请求可以上传多个文件)
spring.servlet.multipart.max-request-size=100MB
```



# 响应

### **@ResponseBody**

类型：**方法注解、类注解**

位置：Controller方法上/类上

作用：将方法返回值直接响应，如果返回值类型是实体对象/集合 ，将会转换为JSON格式响应

说明：`@RestController` = `@Controller` + `@ResponseBody` 

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        String s="Hello World!";
        System.out.println(s);
        return s;
    }
}
```

### 统一响应结果

```java
public class Result {
    //响应码，1 代表成功; 0 代表失败
    private Integer code;
    //提示信息
    private String msg;
    //返回的数据
    private Object data; 
    //…….
}
```



# 分层解耦

## 三层架构

- controller：控制层，接收前端发送的请求，对请求进行处理，并响应数据。

- service：业务逻辑层，处理具体的业务逻辑。

- dao：数据访问层(Data Access Object)（持久层），负责数据访问操作，包括数据的增、删、改、查。

### 概念

**控制反转：** Inversion Of Control，简称IOC。对象的创建控制权由程序自身转移到外部（容器），这种思想称为控制反转。

**依赖注入：** Dependency Injection，简称DI。容器为应用程序提供运行时，所依赖的资源，称之为依赖注入。

**Bean对象：**IOC容器中创建、管理的对象，称之为bean。

### 控制反转

| 注解        | 说明                 | 位置                                            |
| ----------- | -------------------- | ----------------------------------------------- |
| @Component  | 声明bean的基础注解   | 不属于以下三类时，用此注解                      |
| @Controller | @Component的衍生注解 | 标注在控制器类上                                |
| @Service    | @Component的衍生注解 | 标注在业务类上                                  |
| @Repository | @Component的衍生注解 | 标注在数据访问类上（由于与mybatis整合，用的少） |

注：
	声明bean的时候，可以通过value属性指定bean的名字，如果没有指定，默认为类名首字母小写。
	使用以上四个注解都可以声明bean，但是在springboot集成web开发中，声明控制器bean只能用@Controller。

### 依赖注入

#### @Autowired

默认是按照**类型**进行，如果存在多个相同类型的bean，将会报错

如果同类型的bean存在多个：

- `@Primary`

- `@Autowired` + `@Qualifier("bean的名称")`

- `@Resource(name="bean的名称")`

#### `@Resource` 与 `@Autowired`区别

@Autowired 是spring框架提供的注解，而@Resource是JDK提供的注解。

@Autowired 默认是按照类型注入，而@Resource默认是按照名称注入。



# 开发规范-Restful

## 概念

> REST（REpresentational State Transfer），表述性状态转换，它是一种软件架构风格

- REST是风格，是约定方式，约定不是规定，可以打破。
- 描述模块的功能通常使用复数，也就是加s的格式来描述，表示此类资源，而非单个资源。如：users、emps、books…

传统风格

```url
http://localhost:8080/user/getById?id=1     GET：查询id为1的用户
http://localhost:8080/user/saveUser         POST：新增用户
http://localhost:8080/user/updateUser       POST：修改用户
http://localhost:8080/user/deleteUser?id=1  GET：删除id为1的用户
```

REST风格

```url
http://localhost:8080/users/1  GET：查询id为1的用户
http://localhost:8080/users    POST：新增用户
http://localhost:8080/users    PUT：修改用户
http://localhost:8080/users/1  DELETE：删除id为1的用户
```

## 注解

| 注解           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| `@GetMapping`  | 是一个组合注解， 通常用来处理get请求，常用于执行查询操作。是`@RequestMapping(value="这里写的是请求的路径",method = RequestMethod.GET)`的缩写。 |
| @PutMapping    | 是一个组合注解， 通常用来处理post请求，常用于执行添加操作。是`@RequestMapping(value="这里写的是请求的路径",method = RequestMethod.POST)`的缩写。 |
| @PostMapping   | 是一个组合注解，通常用来处理put请求，常用于执行更新操作。是`@RequestMapping(value="这里写的是请求的路径",method = RequestMethod.PUT)`的缩写。 |
| @DeleteMapping | 是一个组合注解。通常用来处理delete请求，常用于执行删除操作。是`@RequestMapping(value="这里写的是请求的路径",method = RequestMethod.DELETE)`的缩写。 |



# 日志打印

@Slf4j

等价于`private static final Logger logger = LoggerFactory.getLogger(this.XXX.class);`

```java
@Slf4j
@RequestMapping("/depts")
@RestController
public class DeptController {
    @GetMapping
    public Result list() {
        log.info("查询全部部门信息");
        return Result.success();
    }
}
```

配置文件

​		在resources文件夹下生成log4j.properties文件（创建的maven工程可能没有resources文件夹，那就创建一个，位置随便，但是一般是放在main文件夹下，和java文件夹同级的位置，需要把resources文件夹标志为source root，在idea编辑器下通过鼠标右键点击就可以）；或者把该文件放在工程目录下，即和src同级的位置，因为该位置是默认的类路径，程序会自动去此处查找log4j.xml或log4j.properties文件。

```properties
### 设置###
log4j.rootLogger = info,stdout,D,E

### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

### 输出DEBUG 级别以上的日志到=logs/error.log ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = logs/log.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 输出ERROR 级别以上的日志到=logs/error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =logs/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

# 配置文件

## XML

```xml
<!--较为臃肿-->
<server>
	<port>8080</port>
	<address>127.0.0.1</address>
</server>
```

## properties

```properties
#层级结构不清晰
server.port=8080
server.address=127.0.0.1
```

## yml/yaml

```yaml
#简洁、数据为中心
server:
   port:  8080
   address: 127.0.0.1
```

### yml基本语法

- 大小写敏感
- 数值前边必须有空格，作为分隔符
- 使用缩进表示层级关系，缩进时，不允许使用Tab键，只能用空格（idea中会自动将Tab转换为空格）
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- 表示注释，从这个字符一直到行尾，都会被解析器忽略

### yml数据格式

```yaml
#对象/Map集合
user:
   name: zhangsan
   age: 18
   password: 123456
#数组/List/Set集合：
hobby:
 - java
 - game
 - sport

```

## 其他

SpringBoot 除了支持配置文件属性配置，还支持Java系统属性和命令行参数的方式进行属性配置。

- Java系统属性

```sh
java -Dserver.port=9000 -jar xxx.jar 
```

- 命令行参数

```sh
java -jar xxx.jar --server.port=10010
```

## 优先级（由高到低）

1. 命令行参数（--xxx=xxx）
2. java系统属性（-Dxxx=xxx）
3. application.properties
4. application.yml
5. application.yaml（忽略）

## 属性注入

### @Value

```java
    @Value("${aliyun.oss.endpoint}")
    private String endpoint ;
    @Value("${aliyun.oss.accessKeyId}")
   	private String accessKeyId ;
  	@Value("${aliyun.oss.accessKeySecret}")
    private String accessKeySecret ;
    @Value("${aliyun.oss.bucketName}")
    private String bucketName ;
```

### @ConfigurationProperties

```java
@Data
@Component
@ConfigurationProperties(prefix = "aliyun.oss")
public class AliOSSProperties {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
}
```

### 对比

- 相同点
  - 都是用来注入外部配置的属性的。
- 不同点
  - @Value注解只能一个一个的进行外部属性的注入。
  - @ConfigurationProperties可以批量的将外部的属性配置注入到bean对象的属性中。

# 会话技术

## Cookie

```java
@Slf4j
@RestController
public class SessionController {
    @GetMapping("/test1")
    public Result Cookie1(HttpServletResponse response) {
        response.addCookie(new Cookie("name", "Tom"));
        return Result.success();
    }

    @GetMapping("/test2")
    public Result Cookie2(HttpServletRequest request) {
        Cookie[] cookies = request.getCookies();
        for (Cookie cookie : cookies) {
            if (cookie.getName().equals("name")) {
                log.info(cookie.getName() + " : " + cookie.getValue());
            }
        }
        return Result.success();
    }
}
```

- 优点
  - HTTP协议中支持的技术
- 缺点
  - 移动端APP无法使用Cookie
  - 不安全，用户可以自己禁用Cookie
  - Cookie不能跨域（跨域区分三个维度：协议、IP/域名、端口）

## Session

```java
@Slf4j
@RestController
public class SessionController {
    @GetMapping("/test1")
    public Result Session1(HttpSession session) {
        log.info("HttpSession-1: {}", session.hashCode());
        session.setAttribute("name", "Tom");
        return Result.success();
    }

    @GetMapping("/test2")
    public Result Session2(HttpServletRequest request) {
        HttpSession session = request.getSession();
        log.info("HttpSession-2: {}", session.hashCode());
        String name = (String) session.getAttribute("name");
        log.info("name" + " : " + name);
        return Result.success();
    }
}
```

- 优点
  - 存储在服务端，安全
- 缺点：
  - 服务器集群环境下无法直接使用Session
  - Cookie的缺点

## Cookie与Session区别

​	如果说Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。

## 令牌技术（[JWT](https://jwt.io/)）

```java
@Test
    void getJWT() {
        Key key = Keys.hmacShaKeyFor("----------这里填签名密钥---------".getBytes());
        Map<String, Object> claims = new HashMap<>();
        claims.put("id", 1);
        claims.put("name", "Tom");
        String jwt = Jwts.builder().setClaims(claims)//自定义内容（载荷）
                .signWith(key, SignatureAlgorithm.HS256)//签名算法
                .setExpiration(new Date(System.currentTimeMillis() + 60 * 60 * 1000))//设置有效期为1小时
                .compact();
        System.out.println(jwt);
    }

    @Test
    void parseJWT() {
        String jwt = "eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiVG9tIiwiaWQiOjEsImV4cCI6MTY4MDYyODQ0M30.WpCrc62KDI3mNP8r5_58OHwrxIWecjQDaAo67lXiM-k";
        Key key = Keys.hmacShaKeyFor("----------这里填签名密钥---------".getBytes());
        Claims claims = (Claims) Jwts.parserBuilder().setSigningKey(key).build().parse(jwt).getBody();
        System.out.println(claims);
    }
```



- 优点
  - 支持PC端、移动端
  - 解决集群环境下的认证问题
  - 减轻服务器端存储压力
- 缺点：需要自己实现

# 过滤器-Filter

## 快速入门

1. 定义Filter：定义一个类，实现 Filter 接口，并重写其所有方法。

```java
@Slf4j
@WebFilter(urlPatterns = "/*")
public class LoginFilter implements Filter {
    @Override
    //初始化方法，Web服务器启动，创建Filter时调用，只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
        log.info("init 初始化方法执行");
    }

    @Override
    //拦截到请求时，调用该方法，可调用多次
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        log.info("拦截到请求——放行前逻辑");
        if (true) {
            filterChain.doFilter(servletRequest, servletResponse);
        } else {
            Result.error("error");
            String result = JSONObject.toJSONString(Result.error("error"));
            log.info(result);
            servletResponse.getWriter().write(result);
        }
        log.info("拦截到请求——放行前逻辑");
    }

    @Override
    //销毁方法，服务器关闭时调用，只调用一次
    public void destroy() {
        Filter.super.destroy();
        log.info("destroy 销毁方法执行");
    }
}
```

2. 配置Filter：Filter类上加 @WebFilter 注解，配置拦截资源的路径。引导类上加 @ServletComponentScan 开启Servlet组件支持。

```java
@ServletComponentScan
@SpringBootApplication
public class TliasWebManagementApplication {

    public static void main(String[] args) {
        SpringApplication.run(TliasWebManagementApplication.class, args);
    }

}
```

## Filter拦截路径

| 拦截路径     | urlPatterns值 | 含义                               |
| ------------ | ------------- | ---------------------------------- |
| 拦截具体路径 | /login        | 只有访问 /login 路径时，才会被拦截 |
| 目录拦截     | /emps/*       | 访问/emps下的所有资源，都会被拦截  |
| 拦截所有     | /*            | 访问所有资源，都会被拦截           |

## 过滤器链

**概念**：一个web应用中，配置了多个过滤器，就形成了一个过滤器链

**执行顺序**：执⾏顺序和类名字符排序有关

## 登录校验Filter流程

1. 获取请求url。
2. 判断请求url中是否包含login，如果包含，说明是登录操作，放行。
3. 获取请求头中的令牌（token）。
4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
5. 解析token，如果解析失败，返回错误结果（未登录）。
6. 放行。

# 拦截器-Interceptor

## 快速入门

1. 定义拦截器，实现HandlerInterceptor接口，并重写其所有方法。

```java
@Slf4j
@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    //目标资源方法执行前执行，放回true：放行，返回false：不放行
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("preHandle 方法执行");
        return true;
    }

    @Override
    //目标资源方法执行后执行
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        log.info("postHandle 方法执行");
    }

    @Override
    //视图渲染完毕后执行，最后执行
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        log.info("afterCompletion 方法执行");
    }
}
```

2. 注册拦截器

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private LoginInterceptor loginInterceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor).addPathPatterns("/**").excludePathPatterns("/login");
    }

}
```

## Interceptor拦截路径

| 拦截路径  | 含义                 | 举例                                                |
| --------- | -------------------- | --------------------------------------------------- |
| /*        | 一级路径             | 能匹配/depts，/emps，/login，不能匹配 /depts/1      |
| /**       | 任意级路径           | 能匹配/depts，/depts/1，/depts/1/2                  |
| /depts/*  | /depts下的一级路径   | 能匹配/depts/1，不能匹配/depts/1/2，/depts          |
| /depts/** | /depts下的任意级路径 | 能匹配/depts，/depts/1，/depts/1/2，不能匹配/emps/1 |

## Filter与Interceptor区别

- 接口规范不同：过滤器需要实现Filter接口，而拦截器需要实现HandlerInterceptor接口。

- 拦截范围不同：过滤器Filter会拦截所有的资源，而Interceptor只会拦截Spring环境中的资源。

## 登录校验Interceptor流程

1. 获取请求url。
2. 判断请求url中是否包含login，如果包含，说明是登录操作，放行。
3. 获取请求头中的令牌（token）。
4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
5. 解析token，如果解析失败，返回错误结果（未登录）。
6. 放行。

# 异常处理

- 方案一：在Controller的方法中进行try…catch处理
  - 代码臃肿，不推荐
- 方案二：全局异常处理器
  - 简单、优雅，推荐

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public Result ex(Exception ex){
        ex.printStackTrace();
        return Result.error(" 对不起,操作失败,请联系管理员 ");
    }
}
```

# AOP

## 概念及实现

**AOP**：Aspect Oriented Programming（面向切面编程、面向方面编程），其实就是面向特定方法编程。

**实现方式**：动态代理是面向切面编程最主流的实现。而SpringAOP是Spring框架的高级技术，旨在管理bean对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程。

**连接点**：JoinPoint，可以被AOP控制的方法（暗含方法执行时的相关信息）

**通知**：Advice，指哪些重复的逻辑，也就是共性功能（最终体现为一个方法）

**切入点**：PointCut，匹配连接点的条件，通知仅会在切入点方法执行时被应用

**切面**：Aspect，描述通知与切入点的对应关系（通知+切入点）

**目标对象**：Target，通知所应用的对象

## 快速入门

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**编写AOP程序**

```java
@Component
@Slf4j
@Aspect
public class TimeAspect {
    @Around("execution(* com.example.service.impl.*.*(..))")
    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        Object result = proceedingJoinPoint.proceed();
        long endTime = System.currentTimeMillis();
        log.info(proceedingJoinPoint.getSignature().getName() + "耗时" + (endTime - startTime) + "ms");
        return result;
    }
}
```



## 注解

- **@Aspect**

  - 位置：类定义上方
  - 作用：设置当前类为切面类
  - 说明：一个beans标签中可以配置多个aop:config标签

- **@Pointcut**

  - 位置：方法定义上方
  - 作用：使用当前方法名作为切入点引用名称
  - 说明：被修饰的方法忽略其业务功能，格式设定为无参无返回值的方法，方法体内空实现（非抽象）

- **@Before**

  - 位置：方法定义上方
  - 作用：标注当前方法作为前置通知
  - 特殊参数：
    - 无

- **@After**

  - 位置：方法定义上方
  - 作用：标注当前方法作为后置通知
  - 特殊参数：
    - 无

- **@AfterReturning**

  - 位置：方法定义上方

  - 作用：标注当前方法作为返回后通知

  - 特殊参数：

    - returning ：设定使用通知方法参数接收返回值的变量名

- **@AfterThrowing**

  - 位置：方法定义上方


    - 作用：标注当前方法作为异常后通知


  - 特殊参数：
      - throwing ：设定使用通知方法参数接收原始方法中抛出的异常对象名

- **@Around**

  - 位置：方法定义上方


    - 作用：标注当前方法作为环绕通知


    - 特殊参数：
    
      - 无

```java
@Component
@Aspect
@Slf4j
public class AspectDemo {
    @Pointcut("execution(* com.example.service.impl.*.*(..))")
    public void pt(){}

    @Before("pt()")
    public void before(){
        log.info("before 方法执行");
    }

    @After("pt()")
    public void after(){
        log.info("after 方法执行");
    }

    @Around("pt()")
    public Object Around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        log.info("around before 方法执行");
        Object result=proceedingJoinPoint.proceed();
        log.info("around after 方法执行");
        return result;
    }

    @AfterReturning("pt()")
    public void afterReturning(){
        log.info("after returning 方法执行");
    }
    @AfterThrowing("pt()")
    public void afterThrowing(){
        log.info("after throwing 方法执行");
    }
}
```

**执行顺序**

1. around before
2. before
3. after returning / after throwing
4. after
5. around after(如果出现异常，则该过程不会执行)

## 通知顺序

当有多个切面的切入点都匹配到了目标方法，目标方法运行时，多个通知方法都会被执行。

1. 不同切面类中，默认按照切面类的类名字母排序：
   	目标方法前的通知方法：字母排名靠前的先执行
   	目标方法后的通知方法：字母排名靠前的后执行
2. **@Order**注解方式自定义顺序
          Order中的value默认值是整形的最大值，而spring在对通知排序时，order中的value越小，优先级越高。

```java
@Component
@Slf4j
@Aspect
@Order(3)
class AspectDemo1 {
    @Pointcut("execution(* com.example.service.impl.*.*(..))")
    public void pt() {
    }

    @Before("pt()")
    public void before() {
        log.info("AspectDemo1 中 before 方法执行");
    }
}

@Component
@Slf4j
@Aspect
@Order(2)
class AspectDemo2 {
    @Before("AspectDemo1.pt()")
    public void before() {
        log.info("AspectDemo2 中 before 方法执行");
    }
}

@Component
@Slf4j
@Aspect
@Order(1)
class AspectDemo3 {
    @Before("AspectDemo1.pt()")
    public void before() {
        log.info("AspectDemo3 中 before 方法执行");
    }
}
```

## 切入点表达式

切入点表达式：描述切入点方法的一种表达式

作用：主要用来决定项目中的哪些方法需要加入通知

### 根据方法的签名来匹配

```java
execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数) throws 异常?)
// 其中带 ? 的表示可以省略的部分
// 访问修饰符：可省略（比如: public、protected）
// 包名.类名： 可省略
// throws 异常：可省略（注意是方法上声明抛出的异常，不是实际抛出的异常）
```

**通配符**

`*`：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分

```java
execution(* com.*.service.*.update*(*))
```

`..` ：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数

```java
execution(* com..DeptService.*(..))
```

**注意事项**

1. 所有业务方法名在命名时尽量规范，方便切入点表达式快速匹配。如：查询类方法都是 find 开头，更新类方法都是 update开头。
2. 描述切入点方法通常基于接口描述，而不是直接描述实现类，增强拓展性。
3. 在满足业务需要的前提下，尽量缩小切入点的匹配范围。如：包名匹配尽量不使用 ..，使用 * 匹配单个包。
4. 根据业务需要，可以使用 且（&&）、或（||）、非（!） 来组合比较复杂的切入点表达式。

## 根据注解匹配

```java
@annotation(注解全类名)
```

1. 创建接口

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLog {
}
```

2. 定义切面类

```java
@Component
@Slf4j
@Aspect
class AspectDemo1 {
    @Pointcut("@annotation(com.example.aspect.MyLog)")
    public void pt() {
    }

    @Before("pt()")
    public void before() {
        log.info("AspectDemo1 中 before 方法执行");
    }

    @After("@annotation(com.example.aspect.MyLog)")
    public void after(){
        log.info("AspectDemo1 中 after 方法执行");
    }
}
```

# Bean

## Bean的获取

### 自动获取

```java
@Autowired
private ApplicationContext applicationContext;
```

### 主动获取

```java
@Autowired
private ApplicationContext applicationContext;

@Test
public void testGetBean() {
//        根据bean的名称获取
    DeptController bean1 = (DeptController) applicationContext.getBean("deptController");
    System.out.println(bean1);

//        根据bean的类型获取
    DeptController bean2 = applicationContext.getBean(DeptController.class);
    System.out.println(bean2);

//        根据bean的名称及类型获取（带类型转换）
    DeptController bean3 = applicationContext.getBean("deptController", DeptController.class);
    System.out.println(bean3);
}
```

## Bean的作用域

| **作用域**  | **说明**                                            |
| ----------- | --------------------------------------------------- |
| singleton   | 容器内同  名称 的 bean 只有一个实例（单例）（默认） |
| prototype   | 每次使用该  bean 时会创建新的实例（非单例）         |
| request     | 每个请求范围内会创建新的实例（web环境中，了解）     |
| session     | 每个会话范围内会创建新的实例（web环境中，了解）     |
| application | 每个应用范围内会创建新的实例（web环境中，了解）     |

通过 @Scope 注解来进行配置作用域

```java
@Scope("prototype")
```

**注意事项**

- 默认singleton的bean，在容器启动时被创建，可以使用`@Lazy`注解来延迟初始化（延迟到第一次使用时）。
- prototype的bean，每一次使用该bean的时候都会创建一个新的实例。
- 实际开发当中，绝大部分的Bean是单例的，也就是说绝大部分Bean不需要配置scope属性。

## 第三方Bean

​	如果要管理的bean对象来自于第三方（不是自定义的），是无法用 @Component 及衍生注解声明bean的，就需要用到 @Bean注解
​	若要管理第三方bean对象，建议对这些bean进行集中分类配置，可以通过 @Configuration 注解声明一个配置类。

1. **在启动类中配置(**不建议)

```java
@SpringBootApplication
public class SpringbootWebConfig2Application {
    @Bean //将方法返回值交给IOC容器管理,成为IOC容器的bean对象
    public SAXReader saxReader(){
        return new SAXReader();
    }
}
```

2. **在配置类中配置**

```java
@Configuration
public class CommonConfig {
    @Bean
    public SAXReader saxReader(){
        return new SAXReader();
    }
}
```

**注意事项**

- 通过@Bean注解的name或value属性可以声明bean的名称，如果不指定，默认bean的名称就是方法名。
- 如果第三方bean需要依赖其它bean对象，直接在bean定义方法中设置形参即可，容器会根据类型自动装配。
- @Component 及衍生注解 与 @Bean注解使用场景？
  - 项目中自定义的，使用@Component及其衍生注解
  - 项目中引入第三方的，使用@Bean注解

# 自动配置

## 方法

1. @ComponentScan 组件扫描

```java
@ComponentScan({"com.example","com.itheima"})
@SpringBootApplication
public class SpringbootWebConfig2Application {
}
```

缺点：使用繁琐，性能低

2. @Import 导入。使用@Import导入的类会被Spring加载到IOC容器中，导入形式主要有以下几种：

   - 导入 普通类

   - 导入 配置类

   - 导入 ImportSelector 接口实现类

   - @EnableXxxx注解，封装@Import注解

## @SpringBootApplication

该注解标识在SpringBoot工程引导类上，是SpringBoot中最最最重要的注解。该注解由三个部分组成：

- @SpringBootConfiguration：该注解与 @Configuration 注解作用相同，用来声明当前也是一个配置类。
- @ComponentScan：组件扫描，默认扫描当前引导类所在包及其子包。
- @EnableAutoConfiguration：SpringBoot实现自动化配置的核心注解。

## @Conditional

- 作用：按照一定的条件进行判断，在满足给定条件后才会注册对应的bean对象到Spring IOC容器中。

- 位置：方法、类

- @Conditional 本身是一个父注解，派生出大量的子注解：

  - @ConditionalOnClass：判断环境中是否有对应字节码文件，才注册bean到IOC容器。

    ```java
    @Bean
    @ConditionalOnClass(name = "io.jsonwebtoken.Jwts") //当前环境存在指定的这个类时，才声明该bean 
    public HeaderParser headerParser(){...}
    ```

    

  - @ConditionalOnMissingBean：判断环境中没有对应的bean（类型 或 名称） ，才注册bean到IOC容器。

    ```java
    @Bean
    @ConditionalOnMissingBean //当不存在当前类型的bean时，才声明该bean 
    public HeaderParser headerParser(){...}
    ```

    

  - @ConditionalOnProperty：判断配置文件中有对应属性和值，才注册bean到IOC容器。

    ```java
    @Bean
    @ConditionalOnProperty(name = "name",havingValue = "itheima") //配置文件中存在对应的属性和值，才注册bean到IOC容器。
    public HeaderParser headerParser(){...}
    ```


# Maven

## 分模块设计与开发

1. 什么是分模块设计? 

   将项目按照功能拆分成若干个子模块

2. 为什么要分模块设计?

   方便项目的管理维护、扩展，也方便模块间的相互调用，资源共享

3. 注意事项

   分模块设计需要先针对模块功能进行设计，再进行编码。不会先将工程开发完毕，然后进行拆分

## 继承与聚合

### 继承

#### 继承关系

- 概念：继承描述的是两个工程间的关系，与java中的继承相似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承。
- 作用：简化依赖配置、统一管理依赖
- 实现：\<parent> … \</parent>
- 打包方式
  - jar：普通模块打包，springboot项目基本都是jar包（内嵌tomcat运行）
  - war：普通web程序打包，需要部署在外部的tomcat服务器中运行
  - pom：父工程或聚合工程，该模块不写代码，仅进行依赖管理


**实现步骤**

1. 创建maven模块 tlias-parent ，该工程为父工程，设置打包方式pom。

   ```xml
   <parent>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-parent</artifactId>
   	<version>2.7.5</version>
   	<relativePath/>
   </parent>
   
   <groupId>com.itheima</groupId>
   <artifactId>tlias-parent</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>pom</packaging>
   ```

2. 在子工程的pom.xml文件中，配置继承关系。

   ```xml
   <groupId>com.itheima</groupId>
   <artifactId>tlias-pojo</artifactId>
   <version>1.0-SNAPSHOT</version>
   <parent>
       <groupId>com.itheima</groupId>
       <artifactId>tlias-parent</artifactId>
       <version>1.0-SNAPSHOT</version>
      <relativePath>../tlias-parent/pom.xml</relativePath>
   </parent>
   ```

3. 在父工程中配置各个工程共有的依赖（子工程会自动继承父工程的依赖）

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.18.24</version>
       </dependency>
   </dependencies>
   ```

**注意事项**

- 在子工程中，配置了继承关系之后，坐标中的groupId是可以省略的，因为会自动继承父工程的 。
- relativePath指定父工程的pom文件的相对位置（如果不指定，将从本地仓库/远程仓库查找该工程）。
- 若父子工程都配置了同一个依赖的不同版本，以子工程的为准。

#### 版本锁定

- 概念：子工程引入依赖时，无需指定 \<version> 版本号，父工程统一管理。变更依赖版本，只需在父工程中统一变更。
- 实现 ：在maven中，可以在父工程的pom文件中通过 \<dependencyManagement> 来统一管理依赖版本。

**实现步骤**

- 父工程

  ```xml
  <dependencyManagement>
      <dependencies>
          <!--JWT令牌-->
          <dependency>
              <groupId>io.jsonwebtoken</groupId>
              <artifactId>jjwt</artifactId>
              <version>0.9.1</version>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

- 子工程

  ```xml
  <dependencies>
      <dependency>
          <groupId>io.jsonwebtoken</groupId>
          <artifactId>jjwt</artifactId>
      </dependency>
  </dependencies>
  ```

#### 自定义属性/引用属性

- 自定义属性

  ```xml
  <properties>
      <lombok.version>1.18.24</lombok.version>
      <jjwt.version>0.9.0</jjwt.version>
  </properties>
  ```

- 引用属性

  ```xml
  <dependencyManagement>
      <dependencies>
          <!--JWT令牌-->
          <dependency>
              <groupId>io.jsonwebtoken</groupId>
              <artifactId>jjwt</artifactId>
              <version>${jjwt.version}</version>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```

  ```xml
  <dependencies>
      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>${lombok.version}</version>
      </dependency>
  </dependencies>
  ```

#### \<dependencyManagement> 与 \<dependencies>区别

- \<dependencies> 是直接依赖,在父工程配置了依赖,子工程会直接继承下来。 
- \<dependencyManagement> 是统一管理依赖版本,不会直接依赖，还需要在子工程中引入所需依赖(无需指定版本)

### 聚合

- 概念：将多个模块组织成一个整体，同时进行项目的构建。
- 聚合工程：一个不具有业务功能的“空”工程（有且仅有一个pom文件）
- 作用：快速构建项目（无需根据依赖关系手动构建，直接在聚合工程上构建即可
- 实现：maven中可以通过 \<modules> 设置当前聚合工程所包含的子模块名称
- 聚合工程中所包含的模块，在构建时，会自动根据模块间的依赖关系设置构建顺序，与聚合工程中模块的配置书写位置无关。

**实现步骤**

```xml
<!--聚合-->
<modules>
    <module>../tlias-pojo</module>
    <module>../tlias-utils</module>
    <module>../tlias-web-management</module>
</modules>
```

### 二者关系

- 作用
  - 聚合用于快速构建项目
  - 继承用于简化依赖配置、统一管理依赖
- 相同点：
  - 聚合与继承的pom.xml文件打包方式均为pom，可以将两种关系制作到同一个pom文件中
  - 聚合与继承均属于设计型模块，并无实际的模块内容
- 不同点：
  - 聚合是在聚合工程中配置关系，聚合可以感知到参与聚合的模块有哪些
  - 继承是在子模块中配置关系，父模块无法感知哪些子模块继承了自己

## Maven私服

### 项目版本

- RELEASE（发行版本）：功能趋于稳定、当前更新，可以用于发行的版本，存储在私服中的RELEASE仓库中。
- SNAPSHOT（快照版本）：功能不稳定、尚处于开停止发中的版本，即快照版本，存储在私服的SNAPSHOT仓库中。

### 私服的使用

使用私服，需要在maven的settings.xml配置文件中，做如下配置：

1. 需要在 **servers** 标签中，配置访问私服的个人凭证(访问的用户名和密码)

   ```xml
   <server>
       <id>maven-releases</id>
       <username>admin</username>
       <password>admin</password>
   </server>
       
   <server>
       <id>maven-snapshots</id>
       <username>admin</username>
       <password>admin</password>
   </server>
   ```

2. 在 **mirrors** 中只配置私服的连接地址(如果之前配置过阿里云，需要直接替换掉)

   ```xml
   <mirror>
       <id>maven-public</id>
       <mirrorOf>*</mirrorOf>
       <url>http://192.168.150.101:8081/repository/maven-public/</url>
   </mirror>
   ```

3.  需要在 **profiles** 中，增加如下配置，来指定snapshot快照版本的依赖，依然允许使用

    ```xml
    <profile>
        <id>allow-snapshots</id>
            <activation>
            	<activeByDefault>true</activeByDefault>
            </activation>
        <repositories>
            <repository>
                <id>maven-public</id>
                <url>http://192.168.150.101:8081/repository/maven-public/</url>
                <releases>
                	<enabled>true</enabled>
                </releases>
                <snapshots>
                	<enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
    </profile>
    ```

4. 如果需要上传自己的项目到私服上，需要在项目的pom.xml文件中，增加如下配置，来配置项目发布的地址(也就是私服的地址)

   ```xml
   <distributionManagement>
       <!-- release版本的发布地址 -->
       <repository>
           <id>maven-releases</id>
           <url>http://192.168.150.101:8081/repository/maven-releases/</url>
       </repository>
       
       <!-- snapshot版本的发布地址 -->
       <snapshotRepository>
           <id>maven-snapshots</id>
           <url>http://192.168.150.101:8081/repository/maven-snapshots/</url>
       </snapshotRepository>
   </distributionManagement>
   ```

5. 发布项目，直接运行 deploy 生命周期即可 (发布时，建议跳过单元测试)
