# 常见缩写

| 缩写 | 原文                        | 含义                                                         |
| ---- | --------------------------- | ------------------------------------------------------------ |
| OOP  | Object Oriented Programming | 面向对象程序设计                                             |
| POJO | Plain Ordinary Java Object  | 简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称 |
| IOC  | Inversion of Control        | 控制反转。它是一种设计思想，由容器将设计好的对象交给容器控制，而非对象内部直接new |
| DI   | Dependency Injection        | 依赖注入。即组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系 |

---

# 常见目录

- bean 【各类数据对象目录】
  -  do(model) 【与数据库表结构一一对应，通过DAO层向上传输数据源对象】
  - dto 【数据传输对象，Service或Manager向外传输的对象】
  - request 【请求传入对象包装】
  - response 【响应输出对象包装】
- common 【共用对象，工具目录】
  - constants 【比如响应码，状态码，各种常数】
  - utils 【工具库】
- dao 【数据连接对象目录】
  - mapper 【mybatis生成的，如果只用mybatis则直接放到dao下，或者dao直接命名成mapper】
  - repository
- config 【配置文件目录】
- controller 【控制器目录】
- service 【服务层目录】
- impl 【服务实现目录】

# 常用依赖

## [fastjson](https://github.com/alibaba/fastjson)

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.26</version>
</dependency>
```

## [jjwt](https://github.com/jwtk/jjwt)

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

## [pageHelper](https://github.com/pagehelper/Mybatis-PageHelper)

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

