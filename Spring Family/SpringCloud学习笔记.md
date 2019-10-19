

# 一、微服务

## 1、微服务

**拆分、独立的进程、独立的数据库**

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自动单独启动或销毁，拥有自己独立的数据库。



## 2、微服务架构

微服务架构是一种架构模式或者说是一种架构风格**，它提倡将单一应用划分成一组小的服务**，每个服务运行在**独立的进程**中，服务之间互相协调、互相配合，为用户提供最终的价值。服务之间采用轻量级的通信机制互相沟通（通常是基于 Http 的 Restful API）

## 3、优缺点

优点：

1）每个服务足够内聚，足够小，代码容易理解这样能聚焦一个指定的业务功能或业务需求

2）开发简单、开发效率提高，一个服务可能就是专一的只干一件事。

3）微服务能够被小团队单独开发，这个小团队是 2 到 5人的开发人员组成

4）微服务是松耦合的，具有功能意义的服务，无论是在开发阶段或部署阶段都是独立的

5）微服务能使用不同的语言开发

6）易于集成

7）易于理解、修改和维护，允许你融合最新技术

**8）微服务只是业务逻辑的代码，不会和 HTML CSS 或其它界面组件混合**：前后端分离

**9）每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一数据库**



缺点：

1）开发人员要处理分布式系统的复杂性

2）多服务的运维难度，随着服务的增加，运维的压力也在增大

3）系统部署依赖

4）服务间通信成本

5）数据一致性

6）系统集成测试

7）性能监控



# 二、SpringCloud入门

SpringCloud = 分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶

基于 SpringBoot 提供了一套微服务解决方案，包括服务注册于发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于 NetFlix 的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

## 1、SpringBoot 与 SpringCloud

SpringBoot：专注于快速方便的开发单个个体微服务

SpringCloud：是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式回话等等集成服务

SpringBoot可以离开 SpringCloud 独立使用开发项目，**但是 SpringCloud 离不开 SpringBoot**，数据依赖关系

**<span style="color:red">SpringBoot 专注于快速、方便的开发单个微服务个体，SpringCloud 关注全局的服务治理框架</span>**

## 2、SpringCloud与Dubbo对比

|              | Dubbo         | SpringCloud                 |
| ------------ | ------------- | --------------------------- |
| 服务注册中心 | Zookeeper     | SpringCloud Netflix Eureka  |
| 服务调用方式 | RPC           | Rest API                    |
| 服务监控     | Dubbo-monitor | SpringBoot Admin            |
| 断路器       | 不完善        | SpringCloud Netflix Hystrix |
| 服务网关     | 无            | SpringCloud Netflix Zuul    |
| 分布式配置   | 无            | SpringCloud Config          |
| 服务跟踪     | 无            | SpringCloud Sleuth          |
| 消息总线     | 无            | SpringCloud Bus             |
| 数据流       | 无            | SpringCloud Stream          |
| 数据流       | 无            | SpringCloud Task            |

**最大区别：SpringCloud抛弃了 Dubbo 的RPC 通信，采用的是基于 Http 的REST方式。**	



# 三、Rest 微服务构建

## 1、创建主 project

microservicecloud

主要是定义 POM 文件，将各个子模块公用的 jar 包等提取出来，类似一个抽象类



## 2、创建 Maven 子module

**<span style="color:red">不要勾选 create from archetype</span>**

子module默认继承父 project 的依赖

1）子工程 microservicecloud-api 生成的 pom.xml 主要包含

```xml
<parent>
    <artifactId>microservicecloud</artifactId>
    <groupId>com.ireadygo.microservicecloud</groupId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>microservicecloud-api</artifactId>
```



**此时父工程 microservicecloud  pom.xml 会添加子工程模块**

```xml
<modules>
    <module>../SSM_Study/microservicecloud/microservicecloud-api</module>
</modules>
```

2）新建部门 Entity 且配合 lombok 使用

```java
@NoArgsConstructor
@Data
@Accessors(chain = true)
public class Dept implements Serializable {

    private Long deptno;    // 主键
    private String dname;   // 部门名称
    private String db_source; // 来自哪个数据库，微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同的数据库

    public Dept(String dname) {
        super();
        this.dname = dname;
    }

    public static void mian(String... args) {
        Dept dept = new Dept()
            .setDname("developer")
            .setDb_source("mysql");
    }
}

```

3）mvn clean，mvn install 后给其它模块应用，达到通用目的。

也即需要用到部门实体的话，不用每个工程都定义一份，直接引用本模块即可



## 3、创建部门微服务提供者 module

**<span style="color:red">约定---》配置---》编码</span>**

新建module：microservicecloud-provider-dept-8001

查看主 project pom.xml

```xml
<modules>
    <module>microservicecloud-api</module>
    <module>microservicecloud-provider-dept-8001</module>
</modules>
```

1）项目配置文件

```yaml
server:
  port: 8001
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis 配置文件所在路径
  type-aliases-package: com.ireadygo.springcloud.entities  # 所有 entity 别名所在路径
  mapper-locations: classpath:mybatis/mapper/**/*.xml       # mapper 映射文件
spring:
  application:
    name: microservicecloud-dept	# 被注册到服务列表中的 application 名
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/clouddb01?serverTimezone=UTC&characterEncoding=utf-8
    username: root
    password: root
```

2）mybatis 配置

mybatis/mybatis.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

3）数据库脚本

```sql
DROP DATABASE IF EXISTS cloudDB01;

CREATE DATABASE cloudDB01 ;
USE cloudDB01;

CREATE TABLE dept
(
	deptno BIGINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
	dname VARCHAR(60),
	db_source VARCHAR(60)
);

INSERT INTO dept(dname,db_source) VALUES('开发部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('人事部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('财务部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('市场部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('运维部',DATABASE());

SELECT * FROM dept;
```

4）部门DAO

```java
@Mapper
public interface DeptDao {

    boolean addDept(Dept dept);

    Dept findById(Long id);

    List<Dept> findAll();
}

```

5）配置 mybatis DAO映射文件

mybatis/mapper/DeptMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ireadygo.springcloud.dao.DeptDao">
    <!-- public boolean addDept(Dept dept); -->
    <insert id="addDept">
        insert into dept(dname,db_source) values (#{dname},DATABASE())
    </insert>

    <!-- public Dept findById(Long id);  如果只有一个参数，可以随便写-->
    <select id="findById" resultType="com.ireadygo.springcloud.entities.Dept">
        select deptno,dname,db_source from dept where id=#{deptno} 
    </select>

    <!-- public List<Dept> findAll(); -->
    <select id="findAll" resultType="com.ireadygo.springcloud.entities.Dept">
        select deptno,dname,db_source from dept
    </select>
</mapper>
```

6）部门 Service 和 ServiceImpl

```java
// 命名没有跟 dao 一致是为了更符合 rest api 风格
public interface DeptService {

    boolean add(Dept dept);

    Dept get(Long id);

    List<Dept> list();
}

@Service
public class DeptServiceImpl implements DeptService {

    @Autowired
    DeptDao deptDao;

    @Override
    public boolean add(Dept dept) {
        return deptDao.addDept(dept);
    }

    @Override
    public Dept get(Long id) {
        return deptDao.findById(id);
    }

    @Override
    public List<Dept> list() {
        return deptDao.findAll();
    }
}

```

7）部门微服务提供者 REST

```java
@RestController
public class DeptController {

    @Autowired
    DeptService deptService;

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept) {
        return deptService.add(dept);
    }

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id) {
        return deptService.get(id);
    }

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list() {
        return deptService.list();
    }
}
```

8）启动测试

```java
@SpringBootApplication
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

测试结果：

```json
[
    {
        "deptno": 1,
        "dname": "开发部",
        "db_source": "clouddb01"
    },
    ......
    {
        "deptno": 5,
        "dname": "运维部",
        "db_source": "clouddb01"
    }
]
```

项目结构：

<img src="Images/SpringCloud/部门微服务提供者.png" style="zoom:80%;" />



## 4、部门微服务消费者 module

microservicecloud-consumer-dept-80

1）配置 pom

```xml
<artifactId>microservicecloud-consumer-dept-80</artifactId>
<description>部门微服务消费者</description>

<dependencies>
    <!-- 引入自定义的 api -->
    <dependency>
        <groupId>com.ireadygo.microservicecloud</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.1.8.RELEASE</version>
    </dependency>
</dependencies>
```

2）项目配置文件

application.yml

```yaml
server:
  port: 80
```

3）SpringBoot 依赖注入注解版配置类

com.ireadygo.springcloud.cfgbean.ConfigBean

```java
@Configuration
public class ConfigBean {

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

4）新建部门服务消费者 rest

com.ireadygo.springcloud.controller.DeptController_Consumer

```java
@RestController
public class DeptController_Consumer {

    private static final String REST_URL_PREFIX = "http://localhost:8001";

    /**
     *  使用restTemplate 访问 restful接口
     *  (url, resultMap, ResponseBean.class)
     *  url：rest请求地址
     *  resultMap：请求参数
     *  ResponseBean：Http响应被转换成的对象
     */

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }

    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(
            REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }

    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(
            REST_URL_PREFIX +"/dept/list", List.class);
    }
}

```

5）启动测试

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}

```



# 四、 Eureka 服务注册与发现

## 1、Eureka 基本架构

Eureka 是 Netflix 的一个子模块，也是核心模块之一。Eureka 是一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了。功能类似于 Dubbo 的注册中心，比如  Zookeeper。

Eureka 采用了 C-S架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心

而系统中的其它微服务，<span style="color:red">**使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。**</span>SpringCloud的一些其它模块（比如 Zuul）就可以通过 Eureka Server来发现系统中的其它微服务，并执行相关的逻辑。

![](Images/SpringCloud/Eureka原理.png)



<span style="color:red">**三大角色**</span>

1）Eureka Server 提供服务注册和发现

2）Service Provider 服务提供方将自身服务注册到 Eureka，从而使服务消费方能够找到

3）Service Consumer 服务消费方从 Eureka 获取注册服务列表，从而能够消费服务



## 2、Eureka Server Module

1）添加SpringCloud依赖

父工程 pom.xml 配置 SpringCloud版本号

注意SpringCloud 与 SpringBoot有兼容问题

参考：https://start.spring.io/actuator/info

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.8.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
</properties>
```

子工程 pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



2）配置 Eureka

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost  # eureka 服务端实例名称
  client:
    register-with-eureka: false   # false  表示不向注册中心注册自己
    fetch-registry: false         # false 表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/  
      # 设置与 Eureka server 交互的地址查询服务和注册服务

```

3）启动 Eureka：**<span style="color:red">@EnableEurekaServer</span>**

**注意由于父工程添加了 mybatis，需要排除 DataSourceAutoConfiguration**

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@EnableEurekaServer
public class EurekaServer7001_App {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}

```



## 3、将客户端注册到 Eureka 服务中

1）修改 microservicecloud-provider-dept-8001 module 的 pom.xml

```xml
<!-- Eureka 客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-config -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```



2）配置客户端向 Eureka 服务端的注册地址

```yaml
eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   # 自定义服务实例ID，Eureka服务列表中status的名称
```



3）开启客户端

```java
@SpringBootApplication
@EnableEurekaClient     // 本服务启动后会自动注册进 Eureka 服务中
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```



## 4、actuator与注册微服务信息完善

1）在客户端 pom.xml文件添加监控

```xml
<!-- 添加监控 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2）主工程添加 Build 构建信息

```xml
<!--主要用于完善监控信息，点击服务跳转 info 页面-->
<build>
    <finalName>microservicecloud</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>

    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>$</delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3）配置客户端 info 页面信息

```yaml
eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   # 自定义服务实例ID
    prefer-ip-address: true     # 显示微服务的 IP 地址

info:
  app.name: moqi-microservicecloud
  company.name: www.ireadygo.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

 客户端配置完整版

```yaml
server:
  port: 8001
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml
  mapper-locations:
    - classpath:mybatis/mapper/**/*.xml
  type-aliases-package: com.ireadygo.springcloud.entities

spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/clouddb01?serverTimezone=UTC&characterEncoding=utf-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   # 自定义服务的名称
    prefer-ip-address: true     # 显示微服务的 IP 地址

info:
  app.name: moqi-microservicecloud
  company.name: www.ireadygo.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```



4）访问服务，进入info页面

```json
{
    "app": {
        "name": "moqi-microservicecloud"
    },
    "company": {
        "name": "www.ireadygo.com"
    },
    "build": {
        "artifactId": "$project.artifactId$",
        "version": "$project.version$"
    }
}
```



## 5、Eureka自动保护机制

某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存

默认情况下，如果 Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络分区故障

**<span style="color:red">在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该 Eureka Server 节点就会自动退出自我保护模式。它的设计哲学额就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着</span>**



## 6、服务发现

服务提供者中实现服务发现：microservicecloud-provider-dept-8001

1）添加请求接口

```java
@Autowired
DiscoveryClient discoveryClient;

@RequestMapping(value = "/dept/discovery", method = RequestMethod.GET)
public Object discovery() {
    List<String> serviceNames = discoveryClient.getServices();
    System.out.println("**************" + serviceNames);
    List<ServiceInstance> serviceInstances 
        = 	discoveryClient.getInstances("microservicecloud-dept");
    for (ServiceInstance serviceInstance : serviceInstances) {
        System.out.println(serviceInstance.getInstanceId() +
                           "\t" + serviceInstance.getHost() + "\t" +
                           serviceInstance.getPort() 
                           + "\t" + serviceInstance.getUri() + "\t");
    }
    return this.discoveryClient;
}
```

2）开启请求

```java
@SpringBootApplication
@EnableEurekaClient     // 本服务启动后会自动注册进 Eureka 服务中
@EnableDiscoveryClient
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

访问地址：http://localhost:8001/dept/discovery



服务消费者发现服务：microservicecloud-consumer-dept-80

```java
@RequestMapping("/consumer/dept/discovery")
public Object discovery() {
    return restTemplate.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
}
```



## 7、集群配置

1）创建集群 module



2）修改 hosts 域名映射

C:\Windows\System32\drivers\etc\hosts

```xml
127.0.0.1	eureka7001.com
127.0.0.1	eureka7002.com
127.0.0.1	eureka7003.com
```

3）修改集群服务的 yml 配置

module：microservicecloud-eureka-7001

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com  # 集群域名配置
  #    hostname: localhost  # eureka 服务端实例名称，单机配置

  client:
    register-with-eureka: false   # false  表示不向注册中心注册自己
    fetch-registry: false         # false 表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  		# eureka 集群配置
      
      #      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/  
      # 单机配置，设置与 Eureka server 交互的地址查询服务和注册服务

```

module：microservicecloud-eureka-7002

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com  # 集群域名配置

  client:
    register-with-eureka: false   # false  表示不向注册中心注册自己
    fetch-registry: false         # false 表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/  # eurekafuwu 集群配置

```

module：microservicecloud-eureka-7003

```yaml
server:
  port: 7003

eureka:
  instance:
    hostname: eureka7003.com  # 集群域名配置

  client:
    register-with-eureka: false   # false  表示不向注册中心注册自己
    fetch-registry: false         # false 表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/  # eurekafuwu 集群配置
```



4）修改集群客户端（服务 provider）配置文件

module：microservicecloud-provider-dept-8001

```yaml
server:
  port: 8001
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml
  type-aliases-package: com.ireadygo.springcloud.entities

spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/clouddb01?serverTimezone=UTC&characterEncoding=utf-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/ # 集群配置
    #      defaultZone: http://localhost:7001/eureka 单机配置

  instance:
    instance-id: microservicecloud-dept8001   # 自定义服务的名称
    prefer-ip-address: true     # 显示微服务的 IP 地址

info:
  app.name: moqi-microservicecloud
  company.name: www.ireadygo.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
  # info build.artifactId 在集群配置中无法解析？
```



## 8、Eureka 与 Zookeeper区别

Netflix在设计Eureka时遵守的就是 <span style="color:red">**AP**</span> 原则

Zookeeper：保证<span style="color:red">**CP**</span>

当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接 down 掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是 zk 会出现这样一种情况，当 master 节点因为网络故障与其它节点失去联系时，剩余节点会重新进行 leader 选举。问题在于，选举 leader 的时间太长，30~120s，且选举期间整个 zk 集群是不可用的，这就导致

在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得 zk 集群失去 master 节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。



Eureka保证AP

Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务

分布式系统必须保证 P：分区容错性

针对并发高 Web 应用一般选择 A：可用性

对于C：一致性，可以选择最终一致性



# 五、Ribbon 负载均衡

## 1、概述

Ribbon提供**客户端的软件**负载均衡算法

1）集中式LB：即在服务的消费方和提供方之间使用独立的 LB 设施（可以是硬件，如F5，也可以是软件，如 Nginx），由该 设施负责把请求通过某种策略转发至服务的提供方；

2）进程内LB

将 LB 逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器

Ribbon属于进程内LB，只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



## 2、配置基本的 Ribbon 负载均衡

1）修改服务消费者 microservicecloud-consumer-dept-80，

 修改pom.xml

```xml
 <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-ribbon -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>

        <!-- Eureka 客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-config -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```



3）修改 RestTemplate，开启负载均衡

```java
@Configuration
public class ConfigBean {

//    开启ribbon负载均衡
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

4）使用服务名访问服务

```java
@RestController
public class DeptController_Consumer {

    //    public static final String REST_URL_PREFIX = "http://localhost:8001"; 不需要端口
    public static final String REST_URL_PREFIX = "http://microservicecloud-dept";

    @Autowired
    RestTemplate restTemplate;

    //    使用Ribbon负载均衡后，会依赖jackson-dataformat-xml这个依赖，导致返回数据为xml
    @RequestMapping(value = "/consumer/dept/get/{id}",
                    produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public Dept get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(
            REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }
    .....
}
```

5）开启客户端

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@EnableEurekaClient
public class DeptConsumer80_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```



6）总结

<span style="color:red">**Ribbon与Eureka整合后，Consumer可以直接调用服务而不用关心地址和端口号**</span>



## 3、Ribbon负载均衡

1）参考microservicecloud-provider-dept-8001 module

新建两个module，8002，8003

修改启动类

```java
@SpringBootApplication
@EnableEurekaClient     // 本服务启动后会自动注册进 Eureka 服务中
@EnableDiscoveryClient
public class DeptProvider8002_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8002_App.class, args);
    }
}

@SpringBootApplication
@EnableEurekaClient     // 本服务启动后会自动注册进 Eureka 服务中
@EnableDiscoveryClient
public class DeptProvider8003_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8003_App.class, args);
    }
}
```

2）拷贝8001的 pom 文件

```xml
<dependencies>
        <!-- 引入自定义的 api 通用包，可以使用 Dept部门 Entity-->
        <dependency>
            <groupId>com.ireadygo.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Eureka 客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-config -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>


    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

3）拷贝 resources目录下的所有资源文件，并修改 application.yml配置文件

```yaml
server:
  port: 8002
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml
  type-aliases-package: com.ireadygo.springcloud.entities

spring:
  application:
    name: microservicecloud-dept  # 被注册到 Eureka 服务端的服务名
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/clouddb02?serverTimezone=UTC&characterEncoding=utf-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/ # 集群配置
    #      defaultZone: http://localhost:7001/eureka 单机配置

  instance:
    instance-id: microservicecloud-dept8002   # 自定义服务实例的ID
    prefer-ip-address: true     # 显示微服务的 IP 地址

info:
  app.name: moqi-microservicecloud
  company.name: www.ireadygo.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

修改端口号：8002

数据库名：clouddb02

服务ID：instance-id: microservicecloud-dept8002 

同理，microservicecloud-provider-dept-8003 也要做相应的修改

4）创建数据库 clouddb02，clouddb03

5）启动 3个 Eureka集群配置

6）启动 3个 Dept 微服务并自测试

7）启动 microservicecloud-consumer-dept-80 

访问：http://localhost/consumer/dept/list

默认依次轮流访问三个Dept服务，实现负载均衡



## 4、核心组件IRule

根据特定算法从服务列表中选取一个要访问的服务

RoundRobinRule：轮询，默认

RandomRule：随机

AvailabilityFilteringRule：会先过滤掉多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问

RetryRule：先按照轮询策略，如果获取服务失败在指定时间内进行重试，获取可用的服务

## 5、自定义Ribbon负载均衡算法





# 六、feign：Rest Client

Feign 是一个声明式的 **Web 服务客户端**，使得编写 Web 服务客户端变得非常容易，只需要创建一个接口，然后在上面添加注解即可。

前面使用 Ribbon+RestTemplate 时，利用 RestTemplate 对 http 请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它（以前是 Dao 接口上面标注 Mapper 注解，现在是一个微服务接口上面标注一个 Feign注解即可），即可完成对服务提供方的接口绑定，简化了使用 SpringCloud Ribbon时，自动封装服务调用客户端的开发量。



**Feign集成了Ribbon**

利用Ribbon维护了 MicroServiceCoud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用



Feign通过接口的方法调用 Rest 服务（之前是 Ribbon+RestTemplate），该请求发送给 Eureka 服务器（http://microservicecloud-dept/dept/list），通过 Feign直接找到服务接口，由于在进行服务调用的时候融合了 Ribbon技术，所以也支持负载均衡作用。



## 1、使用步骤

1）参考 microservicecloud-consumer-dept-80  Module，创建 microservicecloud-consumer-dept-feign   Module

2）修改 pom.xml 添加 openFeign 负载均衡依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

3）修改 microservicecloud-api  Module，增加接口

添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

添加微服务访问接口

```java
// Rest 负载均衡
@FeignClient(value = "microservicecloud-dept")
public interface DeptClientService {

    //    请求服务 Rest 接口的封装
    @RequestMapping(value = "/dept/get/{id}")
    Dept get(@PathVariable("id") Long id);

    @RequestMapping("/dept/add")
    boolean add(Dept dept);

    @RequestMapping(value = "/dept/list")
    List<Dept> list();
}
```

**4）修改 microservicecloud-consumer-dept-feign 客户端 controller**

```java
@RestController
public class DeptController_Consumer_Feign {

    //    集成了 Feign后，使用 DeptClientService 替换 RestTemplate 请求服务
    @Autowired
    DeptClientService deptClientService;

    @RequestMapping(value = "/consumer/dept/get/{id}", 
                    produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public Dept get(@PathVariable("id") Long id) {
        return deptClientService.get(id);
    }

    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        return deptClientService.add(dept);
    }

    @RequestMapping(value = "/consumer/dept/list", 
                    produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public List<Dept> list() {
        return deptClientService.list();
    }

}
```

**对比使用 RestTemplate 实现**

```java
@RestController
public class DeptController_Consumer {

//    public static final String REST_URL_PREFIX = "http://localhost:8001"; 不需要端口
    public static final String REST_URL_PREFIX = "http://microservicecloud-dept";

    @Autowired
    RestTemplate restTemplate;

//    使用Ribbon负载均衡后，会依赖jackson-dataformat-xml这个依赖，导致返回数据为xml
    @RequestMapping(value = "/consumer/dept/get/{id}",
                    produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public Dept get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(
            REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }

    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        return restTemplate.postForObject(
            REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }

    @RequestMapping(value = "/consumer/dept/list", 
                    produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
    }

}
```



5）启用 Feign 负载均衡

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@EnableEurekaClient
@EnableFeignClients
public class DeptConsumer80_feign_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_feign_App.class, args);
    }
}
```



# 七、Hystrix

## 1、服务雪崩

多个微服务之间调用的时候，假设微服务 A 调用微服务 B 和微服务 C，微服务 B 和微服务 C又调用其它的服务，这就是所谓的”扇出“。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的”雪崩效应“。



对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其它系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。



Hystrix是一个用于处理分布式系统的**<span style="color:red">延迟和容错</span>**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超市、异常等，Hystrix能够保证在一个依赖出问题的情况下，<span style="color:red">**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**</span>

”断路器“本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），<span style="color:red">**向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，**</span>这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



## 2、服务熔断：@HystrixCommand 

1）参考 microservicecloud-provider-dept-8001 创建 Module  

microservicecloud-provider-dept-hystrix-8001

2）修改 pom.xml 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

3）编辑 application.yml，修改服务实例名称

```yaml
eureka:
  instance:
    instance-id: microservicecloud-dept8001-hystrix   # 自定义服务的名称
    prefer-ip-address: true     # 显示微服务的 IP 地址
```

4）修改 controller，添加熔断处理

```java
@RequestMapping(value = "/dept/add", method = RequestMethod.POST)
public boolean add(@RequestBody Dept dept) {  // 必须加 @RequestBody
    return deptService.add(dept);
}

@RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
@HystrixCommand(fallbackMethod = "processHystrix_Get")  // 调用服务失败并抛出错误信息后，调用熔断标注的方法
public Dept get(@PathVariable("id") Long id) {
    Dept dept = deptService.get(id);
    if (dept == null) {
        throw new RuntimeException("该ID：" + id + " 没有对应的信息");
    }
    return dept;
}

// 异常处理耦合，可以参考 AOP 机制，使用 Hystrix 服务降级处理
public Dept processHystrix_Get(@PathVariable("id") Long id) {
    return new Dept().setDeptno(id)
        .setDname("该ID："+ id +" 没有对应的信息，Null --- @Hystrix")
        .setDb_source("no this database in MySQL");
}
```

5）开启熔断机制，并测试

```java
@SpringBootApplication
@EnableEurekaClient     // 本服务启动后会自动注册进 Eureka 服务中
@EnableDiscoveryClient
@EnableHystrix  // 开启熔断机制
public class DeptProvider8001_Hystrix_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
    }
}
```



## 3、服务降级：利用 AOP

整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来。

**服务降级处理是在客户端实现完成的，与服务端没有关系**

1）修改 **microservicecloud-api** 工程

根据已有的 DeptClientService 接口新建 FallbackFactory 接口的类 实现类：DeptClientServiceFallbackFactory

```java
// 必须注入容器，统一处理异常，实现服务降级，以后其它的客户端不会调用服务接口
@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(Long id) {
                return new Dept().setDeptno().setDname().setDb_source();
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }

            @Override
            public List<Dept> list() {
                return null;
            }
        };
    }
}

```

2）修改 **microservicecloud-api** 工程，修改 DeptClientService接口的 **@FeignClient** 注解，

添加 **fallbackFactory** 属性

```java
// Rest 负载均衡
//@FeignClient(value = "microservicecloud-dept")
@FeignClient(value = "microservicecloud-dept", fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {
```

3）修改 microservicecloud-consumer-dept-feign 工程，配置 feign.hystrix.enabled=true

```yaml
server:
  port: 80

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/ # 集群配置

feign:
  hystrix:
    enabled: true
```

4）测试

1. 先启动 Eureka 3 个集群：microservicecloud-eureka-7001，microservicecloud-eureka-7002，microservicecloud-eureka-7003

2. 再启动 microservicecloud-provider-dept-8001 微服务

3. 最后启动 microservicecloud-consumer-dept-feign 客户端

4. 正常访问：http://localhost/consumer/dept/get/1

   返回结果：

   **{**"deptno": **1**,"dname": "开发部","db_source": "clouddb01"**}**

   

5. 关闭 microservicecloud-provider-dept-8001 微服务，继续访问

   返回结果：

   **{**"deptno": **1**,"dname": "该ID：1 没有对应的信息，Consumer客户端提供降级信息，此刻服务Provider已经关闭","db_source": "no data in MySQL"**}**



**服务降级，让客户端在服务端不可用时也会获得提示信息而不会挂起耗死服务器**



## 4、服务降级与熔断

**服务熔断：**

一般是某个服务故障或者异常引起，类似现实世界中的”保险丝“，当某个异常条件被触发，

**<span style="color:red">直接熔断整个服务，而不是一直等到此服务超时。</span>**



服务降级：**客户端回调处理**

所谓降级，一般是从整体负荷考虑。就是当某个服务熔断之后，服务器不再被调用，<span style="color:red">**此时客户端可以自己准备一个本地的 fallback 回调，返回一个缺省值**。</span>

这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强。



# 八、Zuul

Zuul包含了对请求的路由和过滤两个最主要的功能：

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础。Zuul和Eureka进行整合，将Zuul自身注册为 Eureka 服务治理下的应用，同时从 Eureka从获得其它微服务的消息，也即以后的访问微服务都是通过 Zuul 跳转后获得。

注意：Zuul 服务最终还是会注册进 Eureka

**提供 = 代理+路由+过滤三大功能**



















# 问题列表

## 1、子工程打包找不到 main 方法

Execution repackage of goal org.springframework.boot:spring-boot-maven-plugin:2.1.8.RELEASE:repackage failed: Unable to find main class

解决方案：

由于多模块打包时找不到main方法，需要在根pom文件中指定mainClass路径

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <mainClass>
                com.ireadygo.microservicecloud.MicroservicecloudApplication
            </mainClass>
        </configuration>
    </plugin>
</plugins>
```

参考：https://stackoverflow.com/questions/23217002/how-do-i-tell-spring-boot-which-main-class-to-use-for-the-executable-jar

## 2、Failed to determine a suitable driver class

该问题是由于在消费者项目中引入了`mybatis-spring-boot-starter`的依赖，这个依赖会根据自动配置约束自己去配置数据源，而消费者项目中并没有`dataSource`相关的配置，所以出错。

解决方案
在消费者工程中移除mybatis-spring-boot-starter依赖
在消费者工程启动时，排除数据源自动配置即可，在SpringBootApplication注解时增加exclude= {DataSourceAutoConfiguration.class}属性，即：
@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})



## 3、Rest 请求实体参数为 null 问题

微服务消费者请求微服务提供者，微服务提供者 controller 中实体参数必须使用 @RequestBody

微服务消费者：Rest请求服务

```java
@Autowired
RestTemplate restTemplate;

@RequestMapping("/consumer/dept/add")
public boolean add(Dept dept) {
    return restTemplate.postForObject(
        REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
}
```

微服务提供者：

```java
@Autowired
DeptService deptService;

@RequestMapping(value = "/dept/add", method = RequestMethod.POST)
public boolean add(@RequestBody  Dept dept) {
    System.out.println("DeptController add dept: "+dept);
    return deptService.add(dept);
}
```



## 4、mapper文件SQL错误导致Spring容器启动失败

```sq
<insert id="addDept">
        insert into dept (dname, db_source)
        values (#{dname}, DATABASE())
</insert>
```



## 5、mapper文件中的映射语句没有写id

There was an unexpected error (type=Internal Server Error, status=500).

Invalid bound statement (not found): com.ireadygo.springcloud.dao.DeptDao.findById

org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.ireadygo.springcloud.dao.DeptDao.findById

```xml
<!--Dept findById(Long id);-->
<select id="findById" resultType="com.ireadygo.springcloud.entities.Dept">
    select *
    from dept
    where deptno = #{deptno}
</select>
```



## 6、微服务 consumer 调用微服务 provider 时，可能产生 4xx 错误，因为消费者就是 Client



## 7、SpringBoot 2.x与 SpringCloud 版本兼容问题

14:44:35.583 [restartedMain] ERROR org.springframework.boot.SpringApplication - Application run failed
java.lang.NoSuchMethodError: org.springframework.boot.builder.SpringApplicationBuilder.<init>([Ljava/lang/Object;)V

兼容列表：

Finchley	兼容Spring Boot 2.0.x，不兼容Spring Boot 1.5.x
Dalston和Edgware	兼容Spring Boot 1.5.x，不兼容Spring Boot 2.0.x
Camden	兼容Spring Boot 1.4.x，也兼容Spring Boot 1.5.x
Brixton	兼容Spring Boot 1.3.x，也兼容Spring Boot 1.4.x
Angel	兼容Spring Boot 1.2.x



## 8 、Eureka 客户端无法注册成功

com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused: connect

```yaml
eureka:
  client:   # 客户端注册进 Eureka 服务列表内
    service-url:
      defualtZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/ # 集群配置
    #      defaultZone: http://localhost:7001/eureka 单机配置
```

<span style="color:red">**defualtZone 写错了：defaultZone**</span>

