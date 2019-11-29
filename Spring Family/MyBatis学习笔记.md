# 一、MyBatis简介

编写SQL ---》**预编译 ---》设置参数 ---》执行SQL** ---》封装结果

框架：整体的解决方案

Hibernate：全自动框架：旨在消除SQL，HQL（支持定制SQL）

**缺点：1）没法对SQL进行定制机优化**

​			**2）数据库字段是全映射，不能只查一个字段**

诉求：SQL语句交给我们开发人员编写，希望SQL不失去灵活性

MyBatis：半自动，轻量级持久化框架

SQL和Java编码分开，功能边界清晰，一个专注业务，一个专注数据

## 1、MyBatis配置文件入门

1）、全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置有优先级顺序的-->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis_study?serverTimezone=UTC&amp;characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!--将我们写好的sql映射文件(EmployeeMapper.xml)一定要注册到全局配置文件中(mybatis-config.xml)-->
    <mappers>
        <mapper resource="mybatis/mapper/EmployeeMapper.xml"/>
    </mappers>

</configuration>
```

2）、mapper 接口

```java
public interface EmployeeMapper {
    Employee getEmpById(Integer id);
}
```

3）、映射文件绑定 mapper 接口

EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ireadygo.mybatis.mapper.EmployeeMapper">
    <select id="getEmpById" resultType="com.ireadygo.mybatis.bean.Employee">
        select * from tbl_employee where id = #{id}
    </select>
</mapper>
```

4）、调用接口

```java
@Test
public void test() throws IOException {
    /**
         * 1、根据xml配置文件（全局配置文件）创建一个 SqlSessionFactory 对象
         *  有数据源及一些运行环境信息
         * 2、sql 映射文件，配置了每一个sql，以及sql的封装规则
         * 3、将sql映射文件注册在全局配置文件中
         * */
    String resource = "mybatis/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = 
        new SqlSessionFactoryBuilder().build(inputStream);

    try (SqlSession openSession = sqlSessionFactory.openSession()) {
        // 获取接口的实现类对象
        // 为接口自动的创建一个代理对象，代理对象执行增删改查操作
        EmployeeMapper employeeMapper = openSession.getMapper(EmployeeMapper.class);
        Employee employee = employeeMapper.getEmpById(1);
        System.out.println(employee);
    }
}
```



## 2、总结

```tiki wiki
1、接口编程
	MyBatis：Mapper  ---》  xxMapper.xml
2、SqlSession代表和数据库的一次会话，用完必须关闭
3、SqlSession 和 Connection 一样都是非线程安全。每次使用都应该去获取新的对象。
4、Mapper接口没有实现类，但是MyBatis会为这个接口生成一个一个代理对象
      （将接口与xml进行绑定）
       EmployeeMapper employeeMapper = openSession.getMapper(EmployeeMapper.class);
5、两个重要的配置文件：
      MyBatis的全局配置文件：包含数据库连接池信息，事务管理器信息等。。。系统运行环境信息
      SQL映射文件：保存了每一个SQL语句的映射信息，将SQL抽取出来
```



# 二、MyBatis全局配置文件

**1、properties**

```xml
<!--
        1、MyBatis可以使用 properties 来引入外部配置文件的内容；
        resource：引入类路径下的资源
        url：引入网络路径或者磁盘路径下的资源
    -->
<properties resource="application.properties"/>

<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```



**2、settings配置**

```xml
<!--2 settings 包含很多重要的配置项-->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
```



3、typeAliases为类名起别名

```xml
<!-- 3、typeAliases：别名处理器，可以为我们的Java类型起别名,别名不区分大小写-->
<typeAliases>
    <!--
        方式1、typeAlias：为某个java类型起别名
            type：指定要起别名的类型全类名；默认别名就是类名小写；employee
            alias：指定新的别名
        -->
    <!--<typeAlias type="com.ireadygo.mybatis.bean.Employee" alias="emp"/>-->

    <!--方式2、 package：为某个包下的所有类批量起别名
                name：指定包名（为当前包及所有子包下的每一个类都起一个默认别名，（类名小写））-->
    <package name="com.ireadygo.mybatis.bean"/>

    <!--方式3、批量起别名的情况下（可能子包有相同名字的类），
			可以使用@Alias注解为某个类型指定新的别名-->
</typeAliases>
```

4、typeHandlers：类型处理器

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。

| `BooleanTypeHandler`    | `java.lang.Boolean`, `boolean` | 数据库兼容的 `BOOLEAN`               |
| ----------------------- | ------------------------------ | ------------------------------------ |
| `ByteTypeHandler`       | `java.lang.Byte`, `byte`       | 数据库兼容的 `NUMERIC` 或 `BYTE`     |
| `ShortTypeHandler`      | `java.lang.Short`, `short`     | 数据库兼容的 `NUMERIC` 或 `SMALLINT` |
| `IntegerTypeHandler`    | `java.lang.Integer`, `int`     | 数据库兼容的 `NUMERIC` 或 `INTEGER`  |
| `LongTypeHandler`       | `java.lang.Long`, `long`       | 数据库兼容的 `NUMERIC` 或 `BIGINT`   |
| `FloatTypeHandler`      | `java.lang.Float`, `float`     | 数据库兼容的 `NUMERIC` 或 `FLOAT`    |
| `DoubleTypeHandler`     | `java.lang.Double`, `double`   | 数据库兼容的 `NUMERIC` 或 `DOUBLE`   |
| `BigDecimalTypeHandler` | `java.math.BigDecimal`         | 数据库兼容的 `NUMERIC` 或 `DECIMAL`  |



5、插件（plugins）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)

- ParameterHandler (getParameterObject, setParameters)

- ResultSetHandler (handleResultSets, handleOutputParameters)

- StatementHandler (prepare, parameterize, batch, update, query)

  

6、环境配置

```xml
<!-- 4、environments：可以配置多种环境，default指定使用某种环境
        environment：配置一个具体的环境信息，必须有两个标签，
        transactionManager：事务管理器，
            type：事务管理器类型，JDBC（JdbcTransactionFactory）| MANAGED（ManagedTransactionFactory）
            自定义事务管理器，实现TransactionFactory接口，type指定为全类名
        dataSource：数据源，
        type：数据源类型，  UNPOOLED（UnpooledDataSourceFactory）
                            |POOLED（PooledDataSourceFactory）
                            |JNDI（JndiDataSourceFactory）
        自定义数据源：实现DataSourceFactory接口，type是全类名
    -->
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>

```

**Configuration.java**

```java
public Configuration() {
    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
    typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);

    typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
    typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
    typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
    ......
}
```



7、数据库厂商标识（databaseIdProvider）

```xml
<!--  5、databaseIdProvider：支持多数据厂商
        type="DB_VENDOR"，VendorDatabaseIdProvider
            作用是得到数据库厂商的标识，MyBatis就能根据数据库厂商标识执行不同的sql
    -->
<databaseIdProvider type="DB_VENDOR">
    <!-- 为不同的数据厂商起别名 -->
    <property name="MYSQL" value="mysql"/>
    <property name="ORACLE" value="oracle"/>
    <property name="SQL SERVER" value="sqlserver"/>
</databaseIdProvider>
```



8、映射文件

```xml
<!--将我们写好的sql映射文件(EmployeeMapper.xml)一定要注册到全局配置文件中(mybatis-config.xml)-->
<!-- 6、mappers：将sql映射注册到全局配置文件中，-->
<mappers>
    <!--
            mapper：注册一个sql映射
                resource：引用类路径下的sql映射文件
                url：引用网络路径或者磁盘路径下的sql映射文件

                注册接口
                class：引用（注册）接口
                    1、有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下
                    2、没有sql映射文件，在接口上使用注解
                    推荐：
                        重要、复杂的Dao接口我们使用sql映射文件方式
                        不重要、简单的Dao接口为了开发快速可以使用注解；
        -->

    <mapper resource="mybatis/mapper/EmployeeMapper.xml"/>
    <!--<mapper class="com.ireadygo.mybatis.mapper.EmployeeMapper"/>-->
    <!--<mapper class="com.ireadygo.mybatis.mapper.EmployeeMapperAnnotation"/>-->

    <!-- 批量注册，使用xml映射文件会报错-->
    <!--<package name="com.ireadygo.mybatis.mapper"/>-->
</mappers>
```

**批量注册时，xml映射文件会报错**



# 三、增删改操作

## 1、基本操作

### 1）、sql映射

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ireadygo.mybatis.mapper.EmployeeMapper">
    <!--<select id="getEmpById" resultType="employee">-->
    <!-- resultType 可以使用别名，但建议使用全类名-->
    <select id="getEmpById" 
            resultType="com.ireadygo.mybatis.bean.Employee" 	
            databaseId="mysql">
        select * from tbl_employee where id = #{id}
    </select>

    <!-- parameterType 可以省略
    获取自增主键的值：
        mysql 支持自增主键，自增主键值的获取，MyBatis也是利用 statement.getGeneratedKeys();
        useGeneratedKeys="true"；使用自增主键获取主键值
        keyProperty：指定对应的主键属性，也就是MyBatis获取到主键值以后，
			将这个值封装给JavaBean的哪个属性
    -->
    <insert id="addEmp" parameterType="com.ireadygo.mybatis.bean.Employee"
            useGeneratedKeys="true" keyProperty="id">
        insert into tbl_employee(last_name,email,gender) 
        values (#{lastName},#{email},#{gender})
    </insert>

    <update id="updateEmp">
        update tbl_employee 
        set last_name=#{lastName}, email=#{email}, gender=#{gender} where id=#{id}
    </update>

    <delete id="deleteEmp">
        delete from tbl_employee where id=#{id}
    </delete>
</mapper>
```

### 2）、测试增删改

```java
/**
     * 测试增删改
     * 1、MyBatis允许增删改直接定义以下类型返回值
     *      Integer、Long、Boolean
     * 2、我们需要手动提交数据
     */
@Test
public void testCURD() throws IOException {
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    try (SqlSession openSession = sqlSessionFactory.openSession()) {
        EmployeeMapper employeeMapper = openSession.getMapper(EmployeeMapper.class);
        // 添加
        // Employee employee = new Employee("小云",0,"444@gamil.com");
        // employeeMapper.addEmp(employee);

        // 修改
        Employee employee = new Employee(3, "小云云", 0, "111@gamil.com");
        Integer result = employeeMapper.updateEmp(employee);
        System.out.println("result: " + result);

        // 删除
        employeeMapper.deleteEmp(1);
        // 手动提交
        openSession.commit();
    }
}
```



# 四、参数处理

## 1、参数分类

1）、单个参数

MyBatis不会做特殊处理

​	#{参数名}：取参数值

2）、多个参数

多个参数会被封装成一个map

**key：param1...paramN**；注意第一个参数为param1，索引从1开始

value：传入的参数值

#{}：就是从map中获取指定key的值

```xml
<select id="getEmpByIdAndLastName" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where id = #{param1} and last_name=#{param2}
</select>
```

3）、命名参数

明确指定封装参数是map的key；@Param("id")

多个参数会被封装成一个map：

​	key：使用@Param注解指定的值

​	value：参数值

​	#{指定的key}取出对应的参数值

```java
Employee getEmpByIdAndLastName(@Param("id") Integer id, @Param("lastName") String lastName);
```

```xml
<select id="getEmpByIdAndLastName" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where id = #{id} and last_name=#{lastName}
</select>
```

4）、POJO

如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入POJO

​	#{属性名}：取出传入的POJO的属性值

5）、Map

如果多个参数不是业务模型中的数据，没有对应的POJO，不经常使用，为了方便，我们也可以传入map

```java
@Test
public void testMapParams() throws IOException {
    /**
         * 1、根据xml配置文件（全局配置文件）创建一个 SqlSessionFactory 对象
         *  有数据源及一些运行环境信息
         * 2、sql 映射文件，配置了每一个sql，以及sql的封装规则
         * 3、将sql映射文件注册在全局配置文件中
         * */
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();

    try (SqlSession openSession = sqlSessionFactory.openSession()) {
        // 获取接口的实现类对象
        // 为接口自动的创建一个代理对象，代理对象执行增删改查操作
        EmployeeMapper employeeMapper = openSession.getMapper(EmployeeMapper.class);

        Map<String,Object> paramMap = new HashMap<>();
        paramMap.put("id",4);
        paramMap.put("lastName","小盈");
        paramMap.put("table","tbl_employee");

        Employee employee = employeeMapper.getEmpByMap(paramMap);
        System.out.println(employee);
    }
}
```

```xml
<select id="getEmpByMap" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from ${table} where id = ${id} and last_name=#{lastName}
</select>
```

**注意表名使用${table}，不能使用#{table}，因为原生JDBC不支持 ? 预编译方式**

6）、TO

如果多个参数不是业务模型中的数据，但是经常要使用，推荐来编写一个TO（Transfer Object）数据传输对象；如分页对象

7）、场景

```java
public Employee getEmp(@Param("id") Integer id, String lastName)
    取值：id=#{id/param1}  lastName=#{param2}

public Employee getEmp(Integer id, @Param("e") Employee emp)
    取值：id=#{param1} lastName=#{param2.lastName/e.lastName}

特别注意：如果是Collection（List、Set）类型或者数组，会把传入的list或者数组封装在map中
key：Collection（collection），如果是List还可以使用这个 key(list)，数组（array）

public Employee getEmpById(List<Integer> ids)
    取值：取出第一个id的值：#{list[0]}
```

## 2、参数源码分析

### 1）、



### 2）、参数值的获取

#{}：可以获取map中的值或者POJO对象属性的值；

${}：可以获取map中的值或者POJO对象属性的值；

```xml
<select id="getEmpByIdAndLastName" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where id = ${id} and last_name=#{lastName}
</select>
```

​				==>  Preparing: select * from tbl_employee where id = 4 and last_name=?

​				==> Parameters: 小盈(String)

区别：

​		#{}：是以预编译的形式，将参数设置到sql语句中；PrepareStatement；

​		${}：取出的值直接拼装在sql语句中；会有安全问题；

​		大多数情况下，我们取参数的值都应该使用 #{}；



​		原生JDBC不支持占位符的地方我们就可以使用 ${} 进行取值

​		比如分表、排序....：按照年份分表拆分

​				select * from  ${year}_salary  where  xxx;

​				select * from tbl_employee order by  ${f_name} ${order}

### 3）、#{} 更丰富的用法

规定参数的规则：

javaType、jdbcType、mode（存储过程）、numericScale、resultMap、typeHandler、jdbcTypeName、expression

jdbcType通常需要在某种特定的条件下被设置：

​	在我们数据为null的时候，有些数据库不能识别MyBatis对null的默认处理。如果Oracle（报错）

​	**MyBatis对 Null 默认映射为 ：JdbcType.OTHER，无效的类型**

​	**但是Oracle不识别 JdbcType.OTHER类型，能识别 Null 类型**

​	MySQL支持 JdbcType.OTHER



​	由于全局配置中：jdbcTypeForNull=OTHER；Oracle不支持

​	1）、#{email, jdbcType=OTHER}；

​	2）、<setting name="jdbcTypeForNull" value="NULL"/>

​	

# 五、映射规则

## 1、resultType

### 1）、select 返回List

```java
List<Employee> getEmpsByLastNameLike(String lastName);
```

```xml
<!-- List<Employee> getEmpsByLastNameLike(String lastName); -->
<!-- resultType如果返回的是一个集合，要写集合中元素的类型 -->
<select id="getEmpsByLastNameLike" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where last_name like #{lastName}
</select>
```

### 2）、select返回一条记录的Map

```java
// 返回一条记录的map：key=列名，value=列的值
Map<String,Object> getEmpByIdReturnMap(Integer id);
```

```xml
<!-- Map<String,Object> getEmpByIdReturnMap(Integer id); -->
<select id="getEmpByIdReturnMap" resultType="map">
    select * from tbl_employee where id = #{id}
</select>
```

返回的map：{gender=0, last_name=小云云, id=3, email=111@gamil.com}

### 3）、select多条记录封装一个map

```java
// 多条记录封装一个map：Map<Integer, Employee> ：键是这条记录的主键，值是记录封装后的JavaBean
// 告诉MyBatis封装这个map的时候使用哪个属性作为map的key
@MapKey("lastName")
Map<Integer,Employee> getEmpByLastNameLikeReturnMap(String lastName);
```

```xml
<!-- public Map<Integer,Employee> getEmpByLastNameLikeReturnMap(String lastName); -->
<select id="getEmpByLastNameLikeReturnMap" 	resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where last_name like #{lastName}
</select>
```

打印结果：{小盈=Employee{id=4, lastName='小盈', gender=0, email='555@gamil.com'},

 小盈盈=Employee{id=5, lastName='小盈盈', gender=0, email='555@gamil.com'}}



## 2、自定义resultMap，实现高级结果映射

### 1）、自定义 column 到 JavaBean 属性的映射

```xml
<!-- 自定义某个JavaBean的封装规则
        type：自定义规则的JavaBean类型
        id：唯一id，方便引用
    -->
<resultMap id="MyEmp" type="com.ireadygo.mybatis.bean.Employee">
    <!-- 指定主键列的封装规则
            column：指定列名
            property：指定column对应的JavaBean属性
        -->
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <!-- 其它不指定的列会自动封装，如果写resultMap推荐把所有的映射规则都写上-->
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
</resultMap>

<!-- resultMap：自定义结果映射规则 -->
<!-- Employee getEmpById(Integer id); -->
<select id="getEmpById" resultMap="MyEmp">
    select * from tbl_employee where id = #{id}
</select>
```

### 2）、场景一：查询员工及员工所在部门信息

```java
Employee getEmpAndDept(Integer id);
```

```xml
<!--
    场景一：
        查询Employee对象的同时查询员工对应的部门
        Employee === Department
        每个员工都有与之对应的部门
    -->
<!-- 方法1：联合查询：级联属性封装结果集-->
<resultMap id="MyDifEmp" type="com.ireadygo.mybatis.bean.Employee">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!-- dept相关列映射，使用了别名 -->
    <result column="did" property="dept.id"/>
    <result column="dept_name" property="dept.departmentName"/>
</resultMap>

<!-- 方法2：使用association定义关联的单个对象的封装规则 -->
<resultMap id="MyDifEmp2" type="com.ireadygo.mybatis.bean.Employee">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!-- association 指定联合的JavaBean对象
            property="dept"：指定哪个属性是联合对象
            javaType：指定联合属性的Java类型（不能省略）
        -->
    <association property="dept" javaType="com.ireadygo.mybatis.bean.Department">
        <id column="did" property="id"/>
        <result column="dept_name" property="departmentName"/>
    </association>
</resultMap>
<!-- Employee getEmpAndDept(Integer id); -->
<select id="getEmpAndDept" resultMap="MyDifEmp">
    SELECT e.`id` id, e.`last_name` last_name, e.`gender` gender, e.`email` email,e.`d_id` d_id,
    d.`id` did, d.`dept_name` dept_name
    FROM tbl_employee e, tbl_dept d
    WHERE e.`d_id` = d.`id` AND e.`id`= #{id};
</select>
```

打印结果：

<==    Columns: id, last_name, gender, email, d_id, did, dept_name
<==        Row: 3, 小云云, 0, 111@gamil.com, 1, 1, 开发部
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
Department{id=1, departmentName='开发部'}



### 3）、分步查询 

1）EmployeeMapperPlus.java

```java
Employee getEmpAndDeptStep(Integer id);
```

2）EmployeeMapperPlus.xml

```xml
<!--使用 association 进行分步查询
        1、先按照员工id查询员工信息
        2、根据查询出的员工信息中的 d_id 值去部门表查出部门信息
        3、部门设置到员工中；
    -->
<!-- Columns: id, last_name, gender, email, d_id -->
<resultMap id="MyEmpStep" type="com.ireadygo.mybatis.bean.Employee">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!-- association定义关联对象的封装规则
            select：表明当前属性调用select指定的方法查出的结果
            did：指定将哪一列的值传给这个方法

            流程：使用select指定的方法（传入column指定列的参数的值）查出的对象，并封装给property属性
         -->
    <association property="dept"
                 select="com.ireadygo.mybatis.mapper.DepartmentMapper.getDeptById" column="d_id"/>
</resultMap>
<!-- Employee getEmpAndDeptStep(Integer id); -->
<select id="getEmpAndDeptStep" resultMap="MyEmpStep">
    select * from tbl_employee where id = #{id}
</select>
```

3）DepartmentMapper.java

```java
Department getDeptById(Integer id);
```

4）DepartmentMapper.xml

```xml
<!-- Department getDeptById(Integer id); -->
<select id="getDeptById" resultType="com.ireadygo.mybatis.bean.Department">
    <!-- 不能使用 * 否则获取dept_name为null -->
    select id, dept_name departmentName from tbl_dept where id = #{id}
</select>
```

打印结果：

Preparing: select * from tbl_employee where id = ? 
==> Parameters: 3(Integer)
<==    Columns: id, last_name, gender, email, **d_id**
<==        Row: 3, 小云云, 0, 111@gamil.com, **1**



====>  Preparing: select id, dept_name departmentName from tbl_dept where id = ? 
====> **Parameters: 1(Integer)**
<====    Columns: id, departmentName
<====        Row: 1, 开发部



### 4）、延迟加载 

```
	可以使用延迟加载（懒加载），按需加载
    Employee ===> Dept；
    我们每次查询Employee对象的时候，都将一起一起查询出来
    部门信息在我们使用的时候再去查询？
    可以在分步查询的基础上加两个配置：
```

```xml
<!-- 显示的指定我们需要更改配置的值，即使是默认的。防止版本更新带来的问题-->
<setting name="lazyLoadingEnabled" value="true"/>
<setting name="aggressiveLazyLoading" value="false"/>
<setting name="lazyLoadTriggerMethods" value=""/>
```

```java
Employee employee = mapperPlus.getEmpAndDeptStep(3);
System.out.println(employee);  // 调用Employee对象的toString方法会触发懒加载，
// 需要配置<setting name="lazyLoadTriggerMethods" value=""/>

//            System.out.println(employee.getDept());
```



### 5）、场景二：查询部门的同时查询部门所有的员工

**方式1：关联查询**

```java
public class Department {

    private Integer id;
    private String departmentName;
    private List<Employee> emps;
```

DepartmentMapper.java

```java
Department getDeptByIdPlus(Integer id);
```



DepartmentMapper.xml

```xml
<!-- Department getDeptById(Integer id); -->
<select id="getDeptById" resultType="com.ireadygo.mybatis.bean.Department">
    <!-- 不能使用 * 否则获取dept_name为null -->
    select id, dept_name departmentName from tbl_dept where id = #{id}
</select>

<!-- collection嵌套结果集的方式，定义关联的集合类型元素的封装规则-->
<!--
    public class Department {
        private Integer id;
        private String departmentName;
        private List<Employee> emps;
    -->
<!-- did  dept_name  ||  eid  last_name  gender  email  -->
<resultMap id="MyDept" type="com.ireadygo.mybatis.bean.Department">
    <id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
    
    <!-- collection定义关联集合类型属性的封装规则
            ofType：指定集合里面元素的类型
         -->
    <collection property="emps" ofType="com.ireadygo.mybatis.bean.Employee">
        <!-- 定义集合中元素的封装规则-->
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="gender" property="gender"/>
        <result column="email" property="email"/>
    </collection>
</resultMap>
<!-- Department getDeptByIdPlus(Integer id); -->
<select id="getDeptByIdPlus" resultMap="MyDept">
    SELECT d.`id` did, d.`dept_name` dept_name,
    e.`id` eid, e.`last_name` last_name, e.`gender` gender, e.`email` email
    FROM tbl_dept d
    LEFT JOIN tbl_employee e
    ON d.`id`= e.`d_id`
    WHERE d.`id` = #{id};
</select>
```

打印结果：

Department{id=1, departmentName='开发部'}
[Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}, Employee{id=4, lastName='小盈', gender=0, email='555@gamil.com'}]



**方式2：分步查询**

DepartmentMapper.java

```java
Department getDeptByIdStep(Integer id);
```

DepartmentMapper.xml

```xml
<resultMap id="MyDeptByStep" type="com.ireadygo.mybatis.bean.Department">
    <id column="id" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <!-- 将tbl_dept表中 id 列传给 getEmpsByDeptId 的参数deptId-->
    <collection property="emps" 
 			    select="com.ireadygo.mybatis.mapper.EmployeeMapperPlus.getEmpsByDeptId" 
                column="{deptId=id}" 
                fetchType="lazy"/>
<!-- Department getDeptByIdStep(Integer id); -->
<select id="getDeptByIdStep" resultMap="MyDeptByStep">
    select id, dept_name departmentName from tbl_dept where id = #{id}
</select>
    
    <!-- 扩展：将多列的值传递给 select
        将多列的值封装成map传递
        column={key1=column1, key2=column2}
        fetchType="lazy"：延迟加载
    -->
```



EmployeeMapperPlus.java

```java
//    Employee getEmpsByDeptId(Integer deptId); // 也可以
List<Employee> getEmpsByDeptId(Integer deptId);
```

EmployeeMapperPlus.xml

```xml
<!-- List<Employee> getEmpsByDeptId(Integer deptId); -->
<select id="getEmpsByDeptId" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where d_id = #{deptId}
</select>
```



### 6）、分步查询中传递多个参数

扩展：将多列的值传递给 select
        将多列的值封装成map传递
        **column={key1=column1, key2=column2}**
        fetchType="lazy"：延迟加载

```xml
<collection property="emps" 
 			    select="com.ireadygo.mybatis.mapper.EmployeeMapperPlus.getEmpsByDeptId" 
                column="{deptId=id}" 
                fetchType="lazy"/>
```



### 7）、鉴别器

```java
Employee getEmpDis(Integer id);
```

```xml
!-- <discriminator javaType="">
    鉴别器：MyBatis可以使用 discriminator 判断某列的值，然后根据某列的值去改变封装行为，封装Employee
    如果查出的是女生：就把部门信息查询出来，否则不查询
    如果是男生，把last_name这一列的值赋给email；
    -->
<resultMap id="MyEmpDis" type="com.ireadygo.mybatis.bean.Employee">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!--
            column：判断指定的列名
            javaType：列值对应的java类型
        -->
    <discriminator javaType="Integer" column="gender">
        <!-- 如果查出的是女生：就把部门信息查询出来，否则不查询 必须指定类型 -->
        <!-- 此处resultType 必须返回Employee -->
        <case value="0" resultType="com.ireadygo.mybatis.bean.Employee">
            <association property="dept"                														select="com.ireadygo.mybatis.mapper.DepartmentMapper.getDeptById" 
                         column="d_id"/>
        </case>
        
        <!-- 如果是男生，把last_name这一列的值赋给email； -->
        <case value="1" resultType="com.ireadygo.mybatis.bean.Employee">
            <id column="id" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="last_name" property="email"/>
            <result column="gender" property="gender"/>
        </case>
    </discriminator>
</resultMap>
<!-- Employee getEmpDis(Integer id); -->
<select id="getEmpDis" resultMap="MyEmpDis">
    select * from tbl_employee where id = #{id}
</select>
```



# 六、动态SQL

## 1、if条件查询

**<if test="id!=null">id=#{id}</if>**

EmployeeMapperDynamicSQL.java

```java
// 携带了哪个字段的查询条件就带上这个字段的值
List<Employee> getEmpByConditionIf(Employee employee);
```

```xml
<!--
        if
        choose (when, otherwise)
        trim (where, set)
        foreach
    -->
<!-- 查询员工，要求，携带了哪个字段的查询条件就带上这个字段的值-->
<!-- Employee getEmpByConditionIf(Employee employee); -->
<select id="getEmpByConditionIf" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee
    <where>
        <!-- 遇到特殊字符应该转义 -->
        <if test="id!=null">id=#{id}</if>
        <!-- 从参数中取值进行判断，拼接语句注意 column = #{param} -->
        <if test="lastName!=null and lastName!=''">and last_name like #{lastName}</if>
        <if test="email!=null and email.trim()!=''">and email = #{email}</if>
        <if test="gender ==0 or gender==1">and gender=#{gender}</if>
    </where>
</select>
```

测试：

```java
EmployeeMapperDynamicSQL dynamicSQL = openSession.getMapper(EmployeeMapperDynamicSQL.class);
Employee employee = new Employee(6,"小林",null,null);
List<Employee> employeeList= dynamicSQL.getEmpByConditionIf(employee);
System.out.println(employeeList);
```

结果：

==>  Preparing: select * from tbl_employee **where id=? and last_name like ?** 
==> Parameters: 6(Integer), 小林(String)



问题：

​	查询的时候如果某些条件没带可能 SQL 拼接会有问题

解决方案：

1、给 where后面加上1==1，以后的条件都 and xxx

**2、MyBatis推荐使用<where>标签将所有查询条件包括在内**

**where标签只能去掉第一个多出来的 and 或者 or**，后面多出的 and 或 or，where不能解决

3、使用 <trim> 可以去掉前后的字符串



## 2、choose分支查询

```java
List<Employee> getEmpByConditionChoose(Employee employee);
```

```xml
<!-- List<Employee> getEmpByConditionChoose(Employee employee); -->
<select id="getEmpByConditionChoose" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee
    <where>
        <!-- 如果带了id就用id查，如果带了lastName就用lastName查；只会进入其中一个-->
        <choose>
            <when test="id!=null">id=#{id}</when>
            <when test="lastName!=null">last_name=#{lastName}</when>
            <when test="email!=null">email=#{email}</when>

            <otherwise>gender=0</otherwise>
        </choose>
    </where>
</select>
```

测试：

```java
// 测试 choose
Employee employee = new Employee(null,null,null,null); // 不带任何条件，进入otherwise
List<Employee> employeeList= dynamicSQL.getEmpByConditionChoose(employee);
System.out.println(employeeList);
```

结果：

Preparing: select * from tbl_employee WHERE **gender=0** 
==> Parameters: 

## 3、set 修改值

```java
void updateEmp(Employee employee);
```

set版本：

```xml
<!-- void updateEmp(Employee employee); -->
<update id="updateEmp">
    update tbl_employee
    <set>
        <if test="lastName != null">last_name = #{lastName},</if>
        <if test="email != null">email = #{email},</if>
        <if test="gender != null">gender = #{gender}</if>
    </set>
    <!-- where 条件要写在 set 外面 -->
    where id=#{id}
</update>
```

trim版本：

prefix：set   // 给 拼串后整个字符串添加一个前缀 **set**

suffixOverrides：，  // **去掉整个字符串后面多余的逗号**

```xml
<!-- void updateEmp(Employee employee); -->
<update id="updateEmp">
    update tbl_employee
    <trim prefix="set" suffixOverrides=","> 
        <if test="lastName != null">last_name = #{lastName},</if>
        <if test="email != null">email = #{email},</if>
        <if test="gender != null">gender = #{gender}</if>
    </trim>
    <!-- where 条件要写在 trim 外面 -->
    where id=#{id}
</update>
```

## 4、foreach

### 1）、遍历集合

```java
 List<Employee> getEmpsByConditionForeach(List<Integer> list1);
```

```xml
<!-- List<Employee> getEmpsByConditionForeach(List<Integer> ids); -->
<!--
        collection：指定要遍历的集合，如果没有指定@Param（"参数名"）,集合名必须为collection或者list
        list类型的参数会特殊处理封装在map中，map的key就叫list
        item：将当期遍历的的元素赋值给指定的变量
        separator：每个元素之间的分隔符
        open：遍历出所有结果拼接一个开始的字符串
        close：遍历出所有结果拼接一个结束的字符串
        index：遍历list时index是索引，item就是当期值
                遍历map时index是map的key，item就是map的value
        #{变量名}就能取出变量的值就也就当期遍历出的元素
    -->
<select id="getEmpsByConditionForeach" resultType="com.ireadygo.mybatis.bean.Employee">
    select * from tbl_employee where id in
    <foreach collection="collection" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
```

### 2）、批量插入

**最好指定集合的参数名**

```java
void addEmps(@Param("empList") List<Employee> emps);
```

**方法1：**MySQL支持，Oracle不支持

```xml
<!-- 批量保存 -->
<!-- void addEmps(List<Employee> emps); -->
<!-- 批量保存 -->
<!--void addEmps(List<Employee> emps); -->
<!--MySQL下批量保存：可以foreach遍历，MySQL支持 values(),(),()语法；Oracle不支持这种批量插入-->
    <insert id="addEmps" >
        insert into tbl_employee(last_name,email,gender,d_id) values
        <foreach collection="empList" item="emp" separator=",">
            (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
        </foreach>
    </insert>
<insert id="addEmps" >
    insert into tbl_employee(last_name,email,gender,d_id) values
    <foreach collection="empList" item="emp" separator=",">
        (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
    </foreach>
</insert>
```

测试：

```java
// 测试 foreach 批量保存
List<Employee> employeeList = new ArrayList<>();
employeeList.add(new Employee("张无忌",1,"aaa@123",new Department(1)));
employeeList.add(new Employee("小龙女",0,"bbb@123",new Department(2)));
dynamicSQLMapper.addEmps(employeeList);

openSession.commit();
```

测试结果：

Preparing: insert into tbl_employee(last_name,email,gender,d_id) **values (?,?,?,?) , (?,?,?,?)** 
==> Parameters: 张无忌(String), aaa@123(String), 1(Integer), 1(Integer), 小龙女(String), bbb@123(String), 0(Integer), 2(Integer)
<==    Updates: 2

**方法2：**

**MySQL需要开启：allowMultiQueries=true**

```properties
jdbc.url=jdbc:mysql://localhost:3306/mybatis_study?serverTimezone=UTC&characterEncoding=utf-8&allowMultiQueries=true
```



```xml
<!-- 这种方式需要数据库连接属性 allowMultiQueries=true
        这种分号分割多个SQL可以用于其它批量（修改、删除）
-->
<insert id="addEmps" >
    <foreach collection="empList" item="emp" separator=";">
        insert into tbl_employee(last_name,email,gender,d_id)
        values (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
    </foreach>
</insert>
```



### 3）、内置参数

```java
List<Employee> getEmpsByInnerParameter(Employee employee);
```

```xml
<!-- 两个内置参数
        不只是方法传递过来的参数可以使用（if）判断，取值
        MyBatis默认还有两个内置参数：
            _parameter：代表整个参数
                单个参数：_parameter就是这个参数
                多个参数：会被封装为一个map；_parameter代表整个map
        _databaseId：如果配置了databaseIdProvider标签
            _databaseId代表当前数据库的别名
    -->
<!-- List<Employee> getEmpsByInnerParameter(Employee employee); -->
<select id="getEmpsByInnerParameter" resultType="com.ireadygo.mybatis.bean.Employee">

    <if test="_databaseId == 'mysql'">
        select * from tbl_employee
        <if test="_parameter != null">
            where id = #{_parameter.id}
        </if>
    </if>
    <if test="_databaseId == 'oracle'">
        select * from tbl_dept
    </if>
</select>
```

### 4）、SQL重用

```xml
<!-- 抽取可重用的sql片段，方便后面引用
        1、经常将要查询的列名，或者插入用的列名抽取出来
        2、include引用sql片段
		3、include还可以自定义一些property，SQL标签内部就可以使用自定义的属性
			，${prop}, 不能使用#{}方式
-->
<sql id="insertColumn">
    <if test="_databaseId == 'mysql'">
        last_name, email, gender, d_id
    </if>
    <if test="_databaseId == 'oracle'">
        last_name, email
    </if>
</sql>
```

```java
<insert id="addEmps" >
    insert into tbl_employee(
    	<include refid="insertColumn"></include>
    ) values
        <foreach collection="empList" item="emp" separator=",">
            (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
    	</foreach>
</insert>
```

include：也可以使用property传递参数给SQL片段



# 七、缓存

## 1、一级缓存

1. 一级缓存：**本地缓存，SqlSession级别的缓存。一级缓存是一直开启的**
              与数据库同一次会话期间查询到的数据会放在本地缓存中
              以后如果需要获取相同的数据，直接从缓存中拿，不需要再去查询数据库

测试：

```java
try (SqlSession openSession = sqlSessionFactory.openSession()) {
    EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

    Employee employee1 = mapper.getEmpById(3);
    System.out.println(employee1);

    Employee employee2 = mapper.getEmpById(3);
    System.out.println(employee2);

    System.out.println(employee1==employee2);
}
```

测试结果：**只有一次查询语句，并且查询的对象相等**

==>  Preparing: select * from tbl_employee where id = ? 
==> Parameters: 3(Integer)
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
true



**一级缓存失效情况（没有使用到当前一级缓存的情况，效果就是，还需要再向数据库发出查询）**
         **1、SQLSession 不同**
         **2、SQLSession 相同，查询条件不同（当前一级缓存还没有这个数据）**
         **3、SQLSession 相同，两次查询之间进行了增删改操作（这次增删改可能对当前数据有影响）**
         **4、SQLSession 相同，手动清除了一级缓存（缓存清空）**

### 1）、1、SQLSession 不同

```java
try (SqlSession openSession = sqlSessionFactory.openSession(); 
     SqlSession otherSession = sqlSessionFactory.openSession();) {
    EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

    Employee employee1 = mapper.getEmpById(3);
    System.out.println(employee1);

    // 使用另外一个SQLSession
    EmployeeMapper otherMapper = otherSession.getMapper(EmployeeMapper.class);
    Employee employee2 = otherMapper.getEmpById(3);
    System.out.println(employee2);

    System.out.println(employee1==employee2);
}
```

**打印结果：因为使用了不同SQLSession，导致一级缓存失效，查询了两次数据库**

**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 3(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}

Opening JDBC Connection
Created connection 728739494.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@2b6faea6]
**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 3(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
**false**



### 2）、SQLSession 相同，查询条件不同

```java
try (SqlSession openSession = sqlSessionFactory.openSession()) {
    EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

    Employee employee1 = mapper.getEmpById(3);
    System.out.println(employee1);

    // SQLSession 相同，查询条件不同
    Employee employee2 = mapper.getEmpById(4);
    System.out.println(employee2);

    System.out.println(employee1==employee2);
}
```

打印结果：

**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 3(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}

**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 4(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 4, 小盈, 0, 555@gamil.com, 1
<==      Total: 1
Employee{id=4, lastName='小盈', gender=0, email='555@gamil.com'}
**false**



### 3）、SQLSession 相同，查询之间进行了增删改操作

```java
try (SqlSession openSession = sqlSessionFactory.openSession()) {
    EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

    Employee employee1 = mapper.getEmpById(3);
    System.out.println(employee1);

    mapper.addEmp(new Employee(null,"test Cache",1,"Cache"));
    openSession.commit();

    // SQLSession 相同，查询之间进行了增删改操作，可能会影响当前查询
    Employee employee2 = mapper.getEmpById(4);
    System.out.println(employee2);

    System.out.println(employee1==employee2);
}
```

打印结果：

**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 3(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
==>  Preparing: insert into tbl_employee(last_name,email,gender) values (?,?,?) 
==> Parameters: test Cache(String), Cache(String), 1(Integer)
<==    Updates: 1
Committing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@101df177]
**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 4(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 4, 小盈, 0, 555@gamil.com, 1
<==      Total: 1
Employee{id=4, lastName='小盈', gender=0, email='555@gamil.com'}
**false**



## 2、二级缓存

**二级缓存：全局缓存，基于namespace级别的缓存，一个namespace对应一个二级缓存；**
工作机制：
**1、一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中；**
**2、如果会话关闭，一级缓存中的数据会被保存到二级缓存中，新的会话查询信息，就可以参照二级缓存中的内容**
**3、SQLSession   EmployeeMapper ---》 Employee**
                              **DepartmentMapper ---》 Department**
**不同的namespace查出的数据会放在自己对应的缓存（map）中**



**查出的数据都会默认先放在一级缓存中，只有会话提交或关闭以后，一级缓存中的数据才会转移到二级缓存中**
**使用：**
### 1）、开启全局二级缓存配置：

```xml
<!-- 开启全局缓存(二级缓存) -->
<setting name="cacheEnabled" value="true"/>
```

### 2）、去 mapper.xml 中配置二级缓存

```xml
<cache eviction="LRU" flushInterval="60000" readOnly="false" size="1024"/>
<!--
        eviction：缓存的回收策略，主要有LRU，FIFO
        flushInterval：缓存刷新间隔，缓存多长时间清空一次，默认false，设置一个毫秒值
        readOnly：默认false
            true：MyBatis认为所有从缓存中获取的数据操作都是只读操作，不会修改数据，为了加快获取速度，直接将缓存数据的引用交给用户。不安全，速度快
            false：MyBatis觉得获取的数据可能会被修改，MyBatis会利用序列化、反序列化的技术克隆一份数据给用户。安全，速度慢
        size：
        type：指定自定义的缓存的全类名，实现Cache接口
    -->
```

### 3）、我们的 POJO 需要实现序列化接口

```java
public class Employee implements Serializable {
```



测试：

```java
try (SqlSession openSession = sqlSessionFactory.openSession(); 
     SqlSession otherSession = sqlSessionFactory.openSession();) {
    EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);

    Employee employee1 = mapper.getEmpById(3);
    System.out.println(employee1);
    openSession.close(); // 需要关闭SQLSession，一级缓存中的数据才会给二级缓存

    // 第二次查询是从二级缓存中拿到的数据，并没有发送新的sql
    EmployeeMapper otherMapper = otherSession.getMapper(EmployeeMapper.class);
    Employee employee2 = otherMapper.getEmpById(3);
    System.out.println(employee2);

    System.out.println(employee1==employee2);
}
```

打印结果：

**Cache Hit Ratio [com.ireadygo.mybatis.mapper.EmployeeMapper]: 0.0**

**==>  Preparing: select * from tbl_employee where id = ?** 
**==> Parameters: 3(Integer)**
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 3, 小云云, 0, 111@gamil.com, 1
<==      Total: 1
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}
.....
**Cache Hit Ratio [com.ireadygo.mybatis.mapper.EmployeeMapper]: 0.5**
Employee{id=3, lastName='小云云', gender=0, email='111@gamil.com'}

**false  // 说明从二级缓存获取的数据是反序列化的**



**只要打印了 Cache Hit Ratio 说明就去二级缓存查询数据了**

根据发送SQL语句的个数就知道是否使用缓存了



### 4）、缓存相关属性

和缓存有关的设置、属性

1）cacheEnabled = true/false：关闭缓存（二级缓存关闭，一级缓存一直可用）

2）每个 select 标签都有 userCache=true

​		userCache=false：一级缓存一直使用，关闭二级缓存

**3）【flushCache="false"：增删改执行完后就会清除缓存，一级，二级缓存全都清空】**

​		**每个增删改标签默认：flushCache="true"**

​		**查询标签默认：flushCache="false"，如果改为true，每次查询都清空缓存**

4）SQLSession.cleanCache()：只清除当前session的一级缓存

5）localCacheScope：本地缓存作用域（一级缓存SESSION）

​		STATEMENT：禁用缓存



**缓存顺序：**

**二级缓存 ---》一级缓存 ---》数据库**

缓存原理：

![](.\Images\MyBatis缓存原理.png)



# 八、整合









# 九、逆向工程

1. A `<jdbcConnection>` element to specify how to connect to the target database
2. A `<javaModelGenerator>` element to specify target package and target project for generated Java model objects
3. A `<sqlMapGenerator>` element to specify target package and target project for generated SQL map files
4. (Optionally) A `<javaClientGenerator>` element to specify target package and target project for generated client interfaces and classes (you may omit the `<javaClientGenerator>`element if you don't wish to generate Java client code)
5. At least one database `<table>` element



# 十、MyBatis运行原理

## 1、运行流程

![](..\Images\MyBatis运行流程.png)

















































常见异常：

Caused by: org.apache.ibatis.reflection.ReflectionException: Error instantiating interface com.ireadygo.mybatis.mapper.EmployeeMapper with invalid types () or values (). Cause: java.lang.NoSuchMethodException: com.ireadygo.mybatis.mapper.EmployeeMapper.<init>()

原因：resultType写成了Mapper接口



Type interface com.ireadygo.mybatis.mapper.DepartmentMapper is not known to the MapperRegistry.

原因：没有注册Mapper映射



Caused by: java.sql.SQLException: SQL String cannot be empty

Mapper映射文件中标签里**没有写 SQL语句**



Caused by: org.apache.ibatis.binding.BindingException: Parameter 'list1' not found. Available parameters are [collection, list]

使用**foreach**时，参数问题