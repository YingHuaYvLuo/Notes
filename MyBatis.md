# 快速入门

application.properties配置文件

```properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=1234
#指定mybatis输出日志的位置,输出控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

mapper接口

```java
@Mapper
public interface UserMapper {
    @Select("select *  from user")
    public List<User> list();
}

```

# Lombok

> Lombok是一个实用的Java类库，能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，并可以自动化生成日志变量，简化java开发、提高效率。

## 注解

| **注解**            | **作用**                                                     |
| ------------------- | ------------------------------------------------------------ |
| @Getter/@Setter     | 为所有的属性提供get/set方法                                  |
| @ToString           | 会给类自动生成易阅读的  toString 方法                        |
| @EqualsAndHashCode  | 根据类所拥有的非静态字段自动重写 equals 方法和  hashCode 方法 |
| @Data               | 提供了更综合的生成代码功能（@Getter  + @Setter + @ToString + @EqualsAndHashCode） |
| @NoArgsConstructor  | 为实体类生成无参的构造器方法                                 |
| @AllArgsConstructor | 为实体类生成除了static修饰的字段之外带有各参数的构造器方法。 |

## 参数占位符

|             | #{...}                                                       | ${...}                                                |
| ----------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| 原理        | 执行SQL时，会将#{…}替换为?，生成预编译SQL，会自动设置参数值。 | 拼接SQL。直接将参数拼接在SQL语句中，存在SQL注入问题。 |
| 使用时机    | 参数传递，都使用#{…}                                         | 如果对表名、列表进行动态设置时使用。                  |
| 存在SQL注入 | 否                                                           | 是                                                    |



# 注解操作

## 删除

SQL语句

```sql
delete from emp where id = 17;
```

接口方法

```java
@Delete("delete from emp where id = #{id}")
public void delete(Integer id);
```

## 新增

SQL语句

```SQL
insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values ('songyuanqiao','宋远桥',1,'1.jpg',2,'2012-10-09',2,'2022-10-01 10:00:00','2022-10-01 10:00:00');
```

接口方法

```java
@Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values(#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
public void insert(Emp emp);

//主键返回
@Options(keyProperty = "id", useGeneratedKeys = true) //会自动将生成的主键值，赋值给emp对象的id属性
@Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values(#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
public void insert(Emp emp);
```

## 修改

SQL语句

```sql
update emp set username = 'songdaxia', name = '宋大侠', gender = 1 , image = '1.jpg' , job = 2, entrydate = '2012-01-01', dept_id = 2, update_time = '2022-10-01 12:12:12' where id = 19;
```

接口方法

```java
@Update("update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}, job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}")
public void update(Emp emp);
```

## 查询

SQL语句

```sql
select * from emp where id = 19;
```

接口方法

```java
@Select("select * from emp where id = #{id}")
public Emp getById(Integer id);
```

字段名与属性名不同解决方法

1. **起别名**：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样。

```java
@Select("select id, username, password, name, gender, image, job, entrydate, dept_id deptId, create_time createTime, update_time updateTime from emp where id = #{id} ")
public Emp getById(Integer id);
```

2. **手动结果映射**：通过 @Results及@Result 进行手动结果映射。

``` java
@Select("select * from emp where id = #{id}")
@Results({
        @Result(column = "dept_id", property = "deptId"),
        @Result(column = "create_time", property = "createTime"),
        @Result(column = "update_time", property = "updateTime")})
public Emp getById(Integer id);
```

3. **开启驼峰命名**：如果字段名与属性名符合驼峰命名规则，mybatis会自动通过驼峰命名规则映射。

```properties
#开启驼峰命名自动映射，即从数据库字段名 a_column 映射到Java 属性名 aColumn。
mybatis.configuration.map-underscore-to-camel-case=true
```

条件查询

SQL语句

```sql
select *  from emp where name like '%张%' and gender = 1 and entrydate between '2010-01-01' and '2020-01-01 ' order by update_time desc;
```

接口方法

```java
//性能低、不安全、存在SQL注入问题
@Select("select * from emp where name like '%${name}%' and gender = #{gender} and entrydate between #{begin} and #{end} order by update_time desc")
public List<Emp> list(String name, Short gender , LocalDate begin , LocalDate end);

//推荐
@Select("select * from emp where name like  concat('%',#{name},'%') and gender = #{gender} and entrydate between #{begin} and #{end} order by update_time desc")
public List<Emp> list(String name, Short gender , LocalDate begin , LocalDate end);
```

# XML映射文件

## 规范

- XML文件的名称与Mapper接口名称一致，并且放置在相同包下（同包同名）。

- XML文件的namespace属性为Mapper接口全限定名一致。

- XML文件中sql语句的id与Mapper 接口中的方法名一致。

## XML文件示例             

```xml
<!--约束-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.EmpXmlMapper">
    <select id="list" resultType="com.example.pojo.Emp">
        select * from emp where name like concat('%',#{name},'%') and gender = #{gender} and entrydate between
        #{beginDate} and #{endDate} order by entrydate DESC
    </select>
    <select id="getById" resultType="com.example.pojo.Emp">
        select * from emp where id = #{id}
    </select>
    <update id="update" >
        update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}, job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}
    </update>
    <insert id="insert" keyProperty="id" useGeneratedKeys="true">
        insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) value (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})
    </insert>
    <delete id="delete">
        delete from emp where id = #{id}
    </delete>
</mapper>
```

# 注解 OR XML

> 使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。
>
> 选择何种方式来配置映射，以及是否应该要统一映射语句定义的形式，完全取决于你和你的团队。 换句话说，永远不要拘泥于一种方式，你可以很轻松地在基于注解和 XML 的语句映射方式间自由移植和切换。
> 																								——[MyBatis中文网](https://mybatis.org/mybatis-3/zh/index.html)

# 动态SQL

> 随着用户的输入或外部条件的变化而变化的SQL语句

## 判断 \<if> \<where> \<set>

- `<if>`：用于判断条件是否成立。使用test属性进行条件判断，如果条件为true，则拼接SQL。

- `<where>`：where 元素只会在子元素有内容的情况下才插入where子句。而且会自动去除子句的开头的AND或OR。
- `<set>`：动态地在行首插入 SET 关键字，并会删掉额外的逗号。（用在update语句中）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.EmpXmlMapper">
    <select id="list" resultType="com.example.pojo.Emp">
        select *
        from emp
        <where>
            <if test="name != null">
                name like concat('%', #{name}, '%')
            </if>
            <if test="gender != null">
                and gender = #{gender}
            </if>
            <if test="beginDate != null and endDate != null">
                and entrydate between #{beginDate} and #{endDate}
            </if>
        </where>
        order by entrydate DESC
    </select>
    <update id="update">
        update emp
        <set>
            <if test="username != null">username=#{username},</if>
            <if test="name != null">name=#{name},</if>
            <if test="gender != null">gender=#{gender},</if>
            <if test="image != null">image=#{image},</if>
            <if test="job != null">job=#{job},</if>
            <if test="entrydate != null">entrydate=#{entrydate},</if>
            <if test="deptId != null">dept_id=#{deptId},</if>
            <if test="updateTime != null">update_time=#{updateTime}</if>
        </set>
        where id = #{id}
    </update>
</mapper>
```

## 遍历循环 \<foreach>

- 属性

  - collection：集合名称

  - item：集合遍历出来的元素/项

  - separator：每一次遍历使用的分隔符

  - open：遍历开始前拼接的片段

  - close：遍历结束后拼接的片段

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.example.mapper.EmpXmlMapper">
      <delete id="deleteByIds">
          delete from emp
          <where>
              <if test="ids != null and ids.size() != 0">
                  id in
                  <foreach collection="ids" item="id" open="(" separator="," close=")">
                      #{id}
                  </foreach>
              </if>
              <if test="ids == null or ids.size() == 0">
                  and 1=2
              </if>
          </where>
      </delete>
  </mapper>
  ```

  

## SQL语句提取  \<sql> \<include>

- `<sql>`：定义可重用的 SQL 片段。
- `<include>`：通过属性refid，指定包含的sql片段。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.EmpXmlMapper">
<!--    <select id="getById" resultType="com.example.pojo.Emp">-->
<!--        select *-->
<!--        from emp-->
<!--        where id = #{id}-->
<!--    </select>-->
    <sql id="commonSelect">
        select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp
    </sql>
     <select id="getById" resultType="com.example.pojo.Emp">
        <include refid="commonSelect"/>
        where id = #{id}
    </select>
</mapper>
```

# 分页查询

## 原始方式

**EmpMapper**

```java
//查询总记录数
@Select("select count(*) from emp")
public Long count();

//分页查询
@Select("select * from emp limit #{start},#{pageSize}")
public List<Emp> page(int start,int end);
```

**EmpServiceImpl**

```java
@Override
public PageBean page(int page, int pageSize){
    long count = empMapper.count();
    int start = (page - 1) * pageSize;
    List<emp> empList = empMapper.page(start, pageSize);
    return new PageBean(count, empList);
}
```

## PageHelper插件

**EmpMapper**

```java
@Select("select * from emp")
public List<Emp> list();
```

**EmpServiceImpl**

```java
@Override
public PageBean page(int page, int pageSize){
    long count = empMapper.count();
    int start = (page - 1) * pageSize;
    List<emp> empList = empMapper.page(start, pageSize);
    return new PageBean(count, empList);
}
```

# 事务管理

## 注解`@Transactional`

**位置**：业务（service）层的方法上、类上、接口上

**作用**：将当前方法交给spring进行事务管理，方法执行前，开启事务；成功执行完毕，提交事务；出现异常，回滚事务

**rollbackFor属性 回滚**

- 默认情况下，只有出现 RuntimeException 才回滚异常。rollbackFor属性用于控制出现何种异常类型，回滚事务。可修改rollbackFor属性为Exception.class，使其对所有异常回滚。

```java
@Transactional(rollbackFor = Exception.class)
```

**propagation属性 传播行为**

- 事务传播行为：指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。

| **属性值**    | **含义**                                                     | **outMethod**                    | **innerMethod**                                              |
| ------------- | ------------------------------------------------------------ | -------------------------------- | ------------------------------------------------------------ |
| REQUIRED      | 【默认值】需要事务，有则加入，无则创建新事务                 | .新开一个Transaction并在其中运行 | .在outMethod的Transaction中运行                              |
| REQUIRES_NEW  | 需要新事务，无论有无，总是创建新事务                         | .新开一个Transaction并在其中运行 | .outMethod的Transaction暂停直至innerMethod中新开的Transaction执行完毕 |
| SUPPORTS      | 支持事务，有则加入，无则在无事务状态中运行                   | .不在Transaction中运行           | .在outMethod的Transaction中运行                              |
| NOT_SUPPORTED | 不支持事务，在无事务状态下运行,如果当前存在已有事务,则挂起当前事务 | .不在Transaction中运行           | .outMethod的Transaction暂停直至innerMethod执行完毕           |
| MANDATORY     | 必须有事务，否则抛异常                                       | .抛出异常                        | .在outMethod的Transaction中运行                              |
| NEVER         | 必须没事务，否则抛异常                                       | .不在Transaction中运行           | .抛出异常                                                    |



## 开始事务管理日志

```yaml
logging:
 level:
  org.springframework.jdbc.support.JdbcTransactionManager: debug
```

