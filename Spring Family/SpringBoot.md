# 一、Spring Boot入门

> ## 2、微服务

微服务：架构风格（服务微化）

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

单体应用：All IN ONE

微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；



@**SpringBootConfiguration**：SpringBoot的配置类

​	标注在某个类上，表示这是一个SpringBoot的配置类；

​	@**Configuration**：配置类上来标注这个注解；

​		配置类----配置文件；配置类也是容器中的一个组件；@Component

@**EnableAutoConfiguration**：开启自动配置功能

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

​	@**AutoConfigurationPackage**：自动配置包

​		@**Import**(AutoConfigurationPackages.Registrar.class)：

​		Spring的底层注解@Import，给容器中导入一个组件；导入的组件由AutoConfigurationPackages.Registrar.class 指定

​		将主配置类（@SpringBootConfiguration 标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器

@**Import**(AutoConfigurationImportSelector.class)

​	给容器中导入组件

​	将所有需要导入的组件以全类名返回，这些组件就会被添加到容器中

​	会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；

![](G:\03_Markdown\Images\自动配置.png)

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

``` Java
SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class，classLoader)
```

SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置导入到容器中，自动配置类就生效，帮我们进行自动配置工作；

J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-2.1.7.RELEASE.jar下面



# 二、配置文件

## 1、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

application.properties

application.yml



配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们自动配置好

## 2、YAML语法

### 1、基本语法

k:(空格)v：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
	port: 8081
	path: /hello
```

属性值也是大小写敏感

### 2、值的写法

字面量：普通的值（数字，字符串，布尔）

​	k: v：字面量直接来写

​		字符串默认不用加上单引号或双引号

​		""： 双引号，不会转义字符串里面的特殊字符，特殊字符会作为本身想表示的意思

​			name："zhangsan \n lisi"：输出；zhangsan 换行 lisi

​		''：单引号，会转义字符，**特殊字符最终只是一个普通的字符串数据**

name：‘zhangsan \n lisi’：输出；zhangsan \n lisi



对象、Map（属性和值）（键值对）：

​	k: v：在下一行来写对象的属性和值的关系；注意缩进

​		对象还是k: v的方式

```yaml
friends:
	lastName: zhangsan
	age: 20
```

行内写法：

```yaml
friends:{lastName: zhangsan, age: 20}
```



数组（List、Set）：

用 -值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets:[cat,dog,pig]
```

## 3、配置文件值注入

配置文件

```yaml
person:
  lastName: zhangsan
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
    - lisi
    - zhaoliu
  dog:
    name: xiaolin
    age: 12
```



JavaBean

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性配置和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *      只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能；
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
```

我们可以导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

### 1、属性文件中文乱码问题

![1566470244465](G:\03_Markdown\Images\1566470244465.png)



### 2、@Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml或者properties都能获取值

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用 @Value；

如果说，我们专门编写了一个JavaBean来和配置文件进行映射，我们就直接使用 @ConfigurationProperties；

### 3、配置文件注入值数据校验

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性配置和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *      只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能；
 *
 *      @ConfigurationProperties 默认从全局获取配置
 */
@Component
@ConfigurationProperties(prefix = "person")
@Validated   // 结合 @Email注解验证
public class Person {
    /**
     * <bean class="Person">
     *     <property name="lastName" value=""字面量/${key}从环境变量、配置文件中获取值/#{SpEL}></property>
     * </bean>
     */

    @Email
    private String lastName;
//    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private Boolean boss;
    private Date birth;
```



### 4、@PropertySource & @ImportResource

@PropertySource加载指定的配置文件

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性配置和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *      只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能；
 *
 *      @ConfigurationProperties 默认从全局获取配置
 */
@Component
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
//@Validated   // 结合 @Email注解验证
public class Person {
    /**
     * <bean class="Person">
     *     <property name="lastName" value=""字面量/${key}从环境变量、配置文件中获取值/#{SpEL}></property>
     * </bean>
     */
//    @Value("${person.last-name}")  // @Value注解不支持@Validated 验证
//    @Email
    private String lastName;
//    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private Boolean boss;
    private Date birth;
```



@ImportResource导入Spring配置文件，让配置文件里的内容生效

SpringBoot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；**@ImportResource** 标注在一个配置类上

```java
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```

SpringBoot不推荐不编写Spring配置文件了；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<bean id="helloService" class="com.ireadygo.springbootsxt.service.HelloService"/>
</beans>
```



SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类===Spring配置文件

2、上使用@Bean给容器中添加组件

```java
/**
 * @Configuration：指明当前是一个配置类，就是来代替之前的Spring配置文件
 * 在配置文件中使用 <bean></bean> 标签添加组件
 */
@Configuration
public class MyAppConfig {

    // 将方法的返回值添加到容器中；方法名默认就是组件的id
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了");
        return new HelloService();
    }
```



## 4、配置文件占位符

### 1、随机数

```java
${random.value},${random.int}
```

### 2、占位符获取之前配置的值，如果没有值，使用：指定默认值



## 5、Profile

### 1、多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yaml

### 2、yml支持多文档块方式

```yaml
server:
  port: 8082
spring:
  profiles:
    active: prod
---
server:
  port: 8083
spring:
  profiles: dev
---
server:
  port: 8084
spring:
  profiles: prod
```



### 3、激活指定profile

1、在配置文件中指定 spring.profiles.active=dev

2、命令行：

java -jar G:\01_SpringBoot_Project\springbootsxt\target\springbootsxt-0.0.1-SNAPSHOT.jar --spring-profile-active=prod

可以直接在测试的时候，配置传入命令行参数

3、虚拟机参数

-Dspring.profile.active=dev



## 6、配置文件加载位置

SpringBoot启动会扫描以下位置的application.properties或者application.yml文件作为SpringBoot默认配置文件

-file:./config/

-file:./

-classpath:/config/

-classpath:/

优先级由高到低，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这个四个位置全部加载主配置文件，**互补配置**；



我们还可以通过spring.config.location改变默认配置文件位置 

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的心位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**



## 7、外部配置加载顺序

**SpringBoot也可以从以下位置加载配置；优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

1.命令行参数

java -jar G:\01_SpringBoot_Project\springbootsxt\target\springbootsxt-0.0.1-SNAPSHOT.jar 

**--server.port=8088**

多个配置用空格分开； --配置项=值

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值



**由jar包外向jar包内进行寻找：**

**优先加载带profile配置**

**6.jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件**

**7.jar包内部的application-{profile}.properties或application.yml（带spring.profile）配置文件**

**8.jar包外部的application.properties或application.yml（不带spring.profile）配置文件**

**9.jar包内部的application.properties或application.yml（不带spring.profile）配置文件**



10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源：

[参考官方文档](<https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-external-config.html>)



## 8、自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](<https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/common-application-properties.html>)

自动配置原理：

1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 **@EnableAutoConfiguration**

2）、@EnableAutoConfiguration作用：

- 利用 **EnableAutoConfigurationImportSelector** 给容器导入一些组件？

- 可以查看selectImports()方法的内容；

- List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置

  ```java
  SpringFactoriesLoader.loadFactoryNames()
  扫描所有 jar 包类路径下 META-INF/spring.factories
  把扫描到的这些文件的内容包装成properties对象
  从properties中获取到 EnableAutoConfiguration.class 类（类名）对应的值，然后把它们添加到容器中
  ```



将类路径下 **META-INF/spring.factories** 里面配置的所有 **EnableAutoConfiguration** 的值加入到了容器中

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

每一个这样的 **xxxAutoConfiguration** 类都是容器中的一个组件，都加入到容器中；用它们来做自动配置；

3）、每一个自动配置类进行自动配置功能

4）、以 **HttpEncodingAutoConfiguration** **（Http编码自动配置）**为例解释自动配置原理

```java
@Configuration	// 表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpProperties.class) // 启动指定类的ConfigurationProperties功能；将配置文件中对应的值和 HttpProperties绑定起来；并把
HttpProperties加入到容器中

@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)//Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class) // 判断当前项目有没有这个类
CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) // 判断配置文件中是否存在某个配置，spring.http.encoding.enabled；如果不存在，判断也是成立的；即使我们配置文件中不配置此属性，也是默认生效的；
public class HttpEncodingAutoConfiguration {

	// 它已经和SpringBoot的配置文件映射了
	private final HttpProperties.Encoding properties;
	
	// 只有一个有参构造器的情况下，参数的值就会从容器中拿
	public HttpEncodingAutoConfiguration(HttpProperties properties) {
		this.properties = properties.getEncoding();
	}
	
	@Bean // 给容器中添加一个组件，这个组件的某些值需要从 properties中获取
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效？

一旦这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的 **properties** 类中获取的，这些类里面的每一个属性又是和配置文件绑定的；



**精髓：**

​	**1）、 SpringBoot启动会加载大量的自动配置类**

​	**2）、看我们需要的功能SpringBoot是否提供了默认配置类**

​	**3）、我们再来看这个配置类中到底配置了哪些组件；（只要我们要用的组件已有，我们就不需要再配置了）**

​	**4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**

xxxAutoConfiguration：自动配置类

给容器中添加组件

xxxProperties：封装配置文件中相关属性；



5）、所有在配置文件中能配置的属性都是在 **xxxProperties** 类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "spring.http") // 从配置文件中获取指定的值和bean的属性进行绑定
public class HttpProperties {

```



# 三、日志

## 1、日志框架

| 日志门面（日志的抽象层）                                     | 日志实现                                         |
| ------------------------------------------------------------ | ------------------------------------------------ |
| ~~JCL(Jakarta Commons Logging)~~    SLF4j（Simple Logging Facade for Java） ~~jboss-logging~~ | Log4j   JUL（java.util.logging） Log4j2  Logback |

左边选一个门面（抽象层）、右边选一个实现；

日志门面：SLF4J；

日志实现：Logback；



SpringBoot：底层是Spring框架，Spring框架默认是用JCL；

​	**SpringBoot选用 SLF4j和Logback**



## 2、SLF4j使用

### 1、如何在系统中使用SLF4j

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用抽象层里面的方法；

给系统导入slf2j的jar 和 logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示：

![](G:\03_Markdown\Images\concrete-bindings.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成实现框架自己本身的配置文件；**

## 2、遗留问题

a（slf4j+logback）：Spring（commons-logging）、Hibernate（jboss-logging）

统一日志记录，即使是别的框架和我一起统一使用 slf4j 进行输出；

![](G:\03_Markdown\Images\legacy.png)

**如何让系统中所有的日志都统一到 slf4j：**

1、将系统中其它日志框架先排除出去；

2、用中间包来替换原有的日志框架；

3、我们导入 slf4j 其它的实现



## 3、SpringBoot日志关系

底层依赖关系

![](G:\03_Markdown\Images\SpringBoot_Log依赖.png)

总结：

​	1）、SpringBoot底层也是使用 slf4j + logback 的方式进行日志记录

​	2）、SpringBoot也把其它的日志都替换成了 slf4j；

​	3）、中间替换包？

```java
private static class Slf4jAdapter {

		public static Log createLocationAwareLog(String name) {
			Logger logger = LoggerFactory.getLogger(name);
			return (logger instanceof LocationAwareLogger ?
					new Slf4jLocationAwareLog((LocationAwareLogger) logger) : new Slf4jLog<>(logger));
		}

		public static Log createLog(String name) {
			return new Slf4jLog<>(LoggerFactory.getLogger(name));
		}
	}
```



​	4）、如果我们要引入其它框架？一定要把这个框架默认的日志依赖移除掉？

​	

**SpringBoot能自动适配所有的日志，而且底层使用 slf4j+logback的方式记录日志，引入其它框架的时候，只需要把这个框架依赖的日志框架排除掉；**



## 4、日志使用

### 1、SpringBoot修改日志的默认配置

```properties
#指定日志的完整路径
#logging.file=G:/springboot.log
#在当前磁盘下创建springboot目录和log目录，使用spring.log作为默认文件
logging.path=G:/springboot/log

# 在控制台输出的日志的格式
logging.pattern.console=
# 在文件中输出的日志的格式
logging.pattern.file=
```



SpringBoot日志默认配置

主jar包下，logging包下logback

base.xml

```xml
<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```

root 默认为 INFO级别

include配置了控制台及文件输出

defaults.xml：规定了默认配置



### 2、指定配置

给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用默认配置了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

logback_spring.xml：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级 Profile 功能

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>
```



# 四、Web开发

## 1、SpringBoot对静态资源的映射规则

```properties
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
// 可以设置和资源有关的参数，缓存时间等
```



```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties
        .getCache()
        .getCachecontrol()
        .toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(
            registry
            .addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .setCachePeriod(getSeconds(cachePeriod))
            .setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(
            registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(
                this.resourceProperties.getStaticLocations()))								.setCachePeriod(getSeconds(cachePeriod))
            .setCacheControl(cacheControl));
    }
}

// 配置欢迎页
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext), 			applicationContext, getWelcomePage(),
        this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors());
    return welcomePageHandlerMapping;
}

//配置喜欢的图标
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
public static class FaviconConfiguration implements ResourceLoaderAware {

    private final ResourceProperties resourceProperties;

    private ResourceLoader resourceLoader;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(resolveFaviconLocations());
        return requestHandler;
    }
```

1）、所有/webjars/**, 都去 classpath:/META-INF/resources/webjars/ 找资源；

​	webjars：以jar包的方式引入静态资源；

2）、”/**“ 访问当前项目的任何资源，（静态资源的文件夹）

```properties
"classpath:/META-INF/resources/",
"classpath:/resources/",  // /resources/resources
"classpath:/static/", 
"classpath:/public/"
"/**"
```

**/java 和 /resources 都为classpath的根路径**

localhost:8080/abc === 去静态资源文件夹里面找abc

![](G:\03_Markdown\Images\静态资源路径.png)

访问js的路径：localhost:8080/asserts/js/Chart.min.js；**注意不用加static，因为本来就是在静态资源路径下找的**

3）、欢迎页、静态资源文件夹下的所有 index.html 页面；被 ”/**“映射；

​	localhost:8080/  找 index 页面

4）、所有的 **/favicon.ico 都是在静态资源文件下找；



## 2、模板引擎

### 1、thymeleaf 使用&语法

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
    
```

只要我们把HTML页面放在 classpath:/templates/, thymeleaf就能自动渲染

使用：

1、导入thymeleaf的名称空间

```xml
xmlns:th="http://www.thymeleaf.org"
```

2、thymeleaf 语法规则

1）、th:text；改变当前元素里面的文本内容；

​	th：任意 html 属性；来替换原生属性的值

![](G:\03_Markdown\Images\thymeleaf语法规则.png)

2）、表达式

```properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}	// 获取变量值；OGNL
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象
    			#ctx: the context object.
                #vars: the context variables.
                #locale: the context locale.
                #request: (only in Web Contexts) the HttpServletRequest object.
                #response: (only in Web Contexts) the HttpServletResponse object.
                #session: (only in Web Contexts) the HttpSession object.
                #servletContext: (only in Web Contexts) the ServletContext object.
            3）、内置的工具对象
            	#execInfo: information about the template being processed.
#messages: methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris: methods for escaping parts of URLs/URIs
#conversions: methods for executing the configured conversion service (if any).
#dates: methods for java.util.Date objects: formatting, component extraction, etc.
#calendars: analogous to #dates, but for java.util.Calendar objects.
#numbers: methods for formatting numeric objects.
#strings: methods for String objects: contains, startsWith, prepending/appending, etc.
#objects: methods for objects in general.
#bools: methods for boolean evaluation.
#arrays: methods for arrays.
#lists: methods for lists.
#sets: methods for sets.
#maps: methods for maps.
#aggregates: methods for creating aggregates on arrays or collections.
#ids: methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

Selection Variable Expressions: *{...}  // 选中表达式和 ${}在功能上一样
    							需要配合<div th:object="${session.user}">使用
    	<div th:object="${session.user}">
            <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
            <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
            <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
        </div>
        
Message Expressions: #{...}  // 获取国际化内容
Link URL Expressions: @{...} //定义URL @{/order/process(execId=${execId},execType='FAST')}

Fragment Expressions: ~{...} // 片段引用表达式 <div th:insert="~{commons :: main}">...</div>
    
Literals
    Text literals: 'one text', 'Another one!',…
    Number literals: 0, 34, 3.0, 12.3,…
    Boolean literals: true, false
    Null literal: null
    Literal tokens: one, sometext, main,…

Text operations:
    String concatenation: +
    Literal substitutions: |The name is ${name}|
    Arithmetic operations:
    Binary operators: +, -, *, /, %
    Minus sign (unary operator): -

Boolean operations:
    Binary operators: and, or
    Boolean negation (unary operator): !, not
    Comparisons and equality:
    Comparators: >, <, >=, <= (gt, lt, ge, le)
    Equality operators: ==, != (eq, ne)
    
Conditional operators:
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)

Special tokens:
    No-Operation: _
```



## 3、SpringMVC自动配置

<https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html>

### 1、Spring MVC Auto-configuration

SpringBoot自动配置好了SpringMVC

以下是SpringBoot对SpringMVC的默认配置

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  自动配置了ViewResolver（视图解析器：根据方法返回值得到视图对象（View），视图对象决定如何渲染（转发/重定向？））

  **ContentNegotiatingViewResolver**：组合所有的视图解析器；

  **如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来**

- Support for serving static resources, including support for WebJars   // 静态资源文件

  

- 自动注册了 `Converter`, `GenericConverter`, and `Formatter` beans.

  Converter：转换器；类型转换使用Converter

  Formatter：格式化器：日期转化

  自己添加的格式化转换器，我们只需要放在容器中即可

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-message-converters)).

  **HttpMessageConverters：SpringMVC用来转换Http请求和响应的；**

  是从容器中确定；自定义的添加到容器就可以使用

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-message-codes)).

- Static `index.html` support.

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-favicon)).

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-web-binding-initializer)).

  我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（添加到容器）

  ```properties
  初始化 WebDataBinder
  请求数据 === JavaBean
  ```

  **org.springframework.boot.autoconfigure.web：web的所有自动场景；**

### 2、扩展SpringMVC

编写一个配置类（@Configuration），是WebMvcConfiguration类型，不能标注@EnableWebMvc 

既保留了所有的自动配置，也能用我们扩展的配置；

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/extendMvc")
                .setViewName("success");
    }
}
```

原理：

​	1）、WebMvcAutoConfiguration是SpringMVC的自动配置类

​	2）、在做其它自动配置时会导入；@Import(EnableWebMvcConfiguration.class)

```java

@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {
    private final WebMvcConfigurerComposite configurers 
        = new WebMvcConfigurerComposite();

    // 从容器中获取所有的 WebMvcConfigurer
    @Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
        }
        // 一个参考实现；将所有的 WebMvcConfigurer 相关的配置都来一起调用；
        //@Override
        //public void addViewControllers(ViewControllerRegistry registry) {
        //	for (WebMvcConfigurer delegate : this.delegates) {
        //		delegate.addViewControllers(registry);
        //	}
        //}
    }
}
```

​	3）、容器中所有的 WebMvcConfigurer 都会起作用；

​	4）、我们的配置类也会被调用；

​	效果：SpringMVC的配置和我们的扩展配置都会起租用；

### 3、全面接管SpringMVC；

SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配；所有的SpringMVC的自动配置都失效了

**我们需要在配置类中添加 @EnableWebMvc**

```java
@Configuration
@EnableWebMvc
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 浏览器发送 /extendMvc 请求来到success页面
        registry.addViewController("/extendMvc")
                .setViewName("success");
    }
}
```

原理：

为什么@EnableWebMvc自动配置就失效了；

1）、@EnableWebMvc的核心

```java
@Import(DelegatingWebMvcConfiguration.class)
	public @interface EnableWebMvc {
}
```

2）、

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

3）、

```java
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
// 容器中没有这个组件的时候，这个自动配置才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

4）、@EnableWebMvc 将 WebMvcConfigurationSupport 组件导入进来；

5）、导入的 WebMvcConfigurationSupport 只是SpringMVC最基本的功能；



## 4、如何修改SpringBoot的默认配置

模式：

​	1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的；如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来

​	2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置

​	3）、xxx Customizer扩展配置

## 5、RestfulCURD

### 2）、国际化

1）、编写国际化配置文件，抽取页面需要的国际化消息；

![](G:\03_Markdown\Images\国际化.png)

2）、SpringBoot自动配置好了管理国际化资源文件的组件；

```java
@EnableConfigurationProperties
public class MessageSourceAutoConfiguration {

	private static final Resource[] NO_RESOURCES = {};

	@Bean
	@ConfigurationProperties(prefix = "spring.messages")
    
    @Bean
	public MessageSource messageSource(MessageSourceProperties properties) {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		if (StringUtils.hasText(properties.getBasename())) {
            // 设置国际化资源文件的基础名（去掉语言国家代码的）
			messageSource.setBasenames(StringUtils
					.commaDelimitedListToStringArray(
                        StringUtils.trimAllWhitespace(properties.getBasename())));
		}
		if (properties.getEncoding() != null) {
			messageSource.setDefaultEncoding(properties.getEncoding().name());
		}
		messageSource.setFallbackToSystemLocale(
            properties.isFallbackToSystemLocale());
		Duration cacheDuration = properties.getCacheDuration();
		if (cacheDuration != null) {
			messageSource.setCacheMillis(cacheDuration.toMillis());
		}
		messageSource.setAlwaysUseMessageFormat(
            properties.isAlwaysUseMessageFormat());
		messageSource.setUseCodeAsDefaultMessage(
            properties.isUseCodeAsDefaultMessage());
		return messageSource;
	}

}


public class MessageSourceProperties {

	/**
	 * Comma-separated list of basenames (essentially a fully-qualified classpath
	 * location), each following the ResourceBundle convention with relaxed 
	 support for
	 * slash based locations. If it doesn't contain a package qualifier (such as
	 * "org.mypackage"), it will be resolved from the classpath root.
	 */
	private String basename = "messages";  
    // 我们的配置文件可以直接放在类路径下叫 messages.properties;
```

```properties
spring.messages.basename=i18n.login
```

3）、去页面获取国际化的值

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="Mark Otto, Jacob Thornton, and Bootstrap contributors">
    <meta name="generator" content="Jekyll v3.8.5">
    <title>Signin Template · Bootstrap</title>

    <!-- Bootstrap core CSS -->
    <!-- 加上 / 的好处是自动带上项目路径名-->
    <link href="../static/css/bootstrap.min.css" th:href="@{/css/bootstrap.min.css}" rel="stylesheet"  crossorigin="anonymous">

    <!-- Custom styles for this template -->
    <link href="../static/css/signin.css" th:href="@{/css/signin.css}" rel="stylesheet">
</head>
<body class="text-center">
<form class="form-signin">
    <img class="mb-4" src="../static/image/bootstrap-solid.svg" th:src="@{/image/bootstrap-solid.svg}" alt="" width="72" height="72">
    <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
    <label class="sr-only" th:text="#{login.username}">Username</label>
    <input type="text"  name="username" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
    <label for="inputPassword" class="sr-only" 
           th:text="#{login.password}">Password</label>
    <input type="password" id="inputPassword" class="form-control" 
           th:placeholder="#{login.password}" placeholder="Password" required>
    <div class="checkbox mb-3">
        <label>
            <input type="checkbox" value="remember-me"> [[#{login.remember}]]
        </label>
    </div>
    <button class="btn btn-lg btn-primary btn-block" type="submit" 
            th:text="#{login.btn}">Sign in</button>
    <p class="mt-5 mb-3 text-muted">&copy; 2017-2019</p>
    <a  class="btn btn-sm">中文</a>
    <a  class="btn btn-sm">English</a>
</form>
</body>
</html>

```

效果：根据浏览器语言设置的信息切换了国际化



原理：

​		国际化 Locale（区域信息对象）；LocaleResolver（获取区域信息对象）

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
    if (this.mvcProperties.getLocaleResolver() 
        == WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }
    AcceptHeaderLocaleResolver localeResolver = 
        new AcceptHeaderLocaleResolver();
    localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
    return localeResolver;
}
默认的就是根据请求头带来的区域信息获取 Locale进行国际化
```

4）、点击链接切换国际化

```java
public class MyLocaleResolver implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("l");
        Locale locale = java.util.Locale.getDefault();
        if (!StringUtils.isEmpty(l)) {
            String[] splits = l.split("_");
            locale = new Locale(splits[0], splits[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, 
                          HttpServletResponse response, Locale locale) {

    }
}

	@Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
```

### 3）、登录

开发期间模板引擎页面修改以后，要实时生效

1）、禁用模板引擎的缓存

```properties
# 禁用缓存
spring.thymeleaf.cache=false
```

2）、页面修改完成以后 Ctrl+F9：重新编译；

登录错误信息提示：

```html
<p style="color: red" th:if="${not #strings.isEmpty(msg)}" th:text="${msg}" ></p>
```

3）、注册拦截器进行登录验证

```java
@PostMapping("/user/login")
public String login(@RequestParam("username") String userName,
                    @RequestParam("password") String password,
                    Map<String, Object> map, HttpSession httpSession) {
    if (!StringUtils.isEmpty(userName) && "123456".equals(password)) {
        // 登录成功，防止表单重复提交，可以重定向到主页
        httpSession.setAttribute("loginUser", userName);
        return "redirect:/main.html";
    } else {
        // 登录失败
        map.put("msg", "用户名密码错误");
        return "login";
    }
}

@Bean // 将组件注册到容器中
public WebMvcConfigurer webMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/index.html").setViewName("login");
            registry.addViewController("/main.html").setViewName("dashboard");
        }

        // 注册拦截器
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LoginHandlerInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/index.html", "/", 
                                     "/user/login", "/asserts/**"); 
            // 不要拦截登录页面，及静态资源
            // 因为static下有多种资源，所以添加一个asserts目录
        }
    };
}
```



400：一般是表单没有相关字段

405：没有权限，可能是后台没有提供功能

### 4）、CURD-员工列表

1）、RestfulCURD：CURD满足Rest风格；

URI：/资源名称/资源标识  HTTP请求方式区分对资源CURD操作



|      | 普通CURD（uri来区分操作） | RestfulCURD       |
| ---- | ------------------------- | ----------------- |
| 查询 | getEmp                    | emp---GET         |
| 添加 | addEmp?xxx                | emp---POST        |
| 修改 | updateEmp?id=xxx&xxx=xx   | emp/{id}---PUT    |
| 删除 | deleteEmp?id=1            | emp/{id}---DELETE |

2）、请求的架构；

|                                      | 请求URI  | 请求方式 |
| ------------------------------------ | -------- | -------- |
| 查询所有员工                         | emps     | GET      |
| 查询某个员工                         | emp/{id} | GET      |
| 来到添加页面                         | emp      | GET      |
| 添加员工                             | emp      | POST     |
| 来到修改页面（查询员工进行信息回显） | emp{id}  | GET      |
| 修改员工                             | emp      | PUT      |
| 删除员工                             | emp/1    | DELETE   |

3）、员工列表

thymeleaf公共页面元素抽取

```html
1.抽取公共片段，片段名没有{}
<div th:fragment="copy">
    &copy; 2011 The Good Thymes Virtual Grocery
</div>

2.引入公共片段
<div th:insert="~{footer :: copy}"></div>
~{templatename::selector} // 模板名::选择器
~{templatename::fragmentname} // 模板名::片段名，模板名前不要加项目跟路径 /

3.默认效果：
insert的功能片段在div标签中
如果使用th:insert等属性进行引入，可以不用写 ~{};
行内写法可以加上：[[~{}]];[(!{})]
```

三种引入功能片段的的th属性：

**th:insert**：将公共片段整个插入到声明引入的元素中

**th:replace**：将声明引入的元素替换为公共片段

**th:include**：将被引入的片段的内容包含金这个标签中

```html
<footer th:fragment="copy">
  &copy; 2011 The Good Thymes Virtual Grocery
</footer>

引入方式
<div th:insert="footer :: copy"></div>
<div th:replace="footer :: copy"></div>
<div th:include="footer :: copy"></div>

效果
<div>
    <footer>
      &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
    &copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
    &copy; 2011 The Good Thymes Virtual Grocery
</div>
  

```

引入片段的时候传入参数：

```html
<div th:replace="commons/bar::#sidebar(activeUri='emps')"></div>

<li class="nav-item">
    <a class="nav-link active" th:href="@{/main.html}"
       th:class=
       "${activeUri == 'main.html'?'nav-link active':'nav-link'}">
        <span data-feather="home"></span>
        Dashboard <span class="sr-only">(current)</span>
    </a>
</li>
```

### 5）、添加员工

```html
<form th:action="@{/emp}" method="post">
    <div class="form-group">
        <label>LastName</label>
        <input name="lastName" type="text" 
               class="form-control" placeholder="zhangsan">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input name="email" type="email" 
               class="form-control" placeholder="zhangsan@atguigu.com">
    </div>
    <div class="form-group">
        <label>Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" 
                   name="gender" value="1">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" 
                   name="gender" value="0">
            <label class="form-check-label">女</label>
        </div>
    </div>
    <div class="form-group">
        <label>department</label>
        <!--提交的是部门的id-->
        <select class="form-control" name="department.id">
            <option th:each="dept : ${depts}" 
                    th:value="${dept.id}" 
                    th:text="${dept.departmentName}">1
            </option>
        </select>
    </div>
    <div class="form-group">
        <label>Birth</label>
        <input name="birth" type="text" 
               class="form-control" placeholder="zhangsan">
    </div>
    <button type="submit" class="btn btn-primary">添加</button>
</form>
```

```java
// 来到员工添加页面
    @GetMapping("/emp")
    public String toAddPage(Model model) {
        // 来到添加页面，查出所有的部门，在页面显示
        Collection<Department> departments = departmentDao.getDepartments();
        model.addAttribute("depts", departments);
        return "emp/add";
    }

    // 员工添加
    // SpringMVC自动将请求参数与入参对象的属性进行--绑定，要求请求参数的名字和javabean入参对象里面的属性名一致
    @PostMapping("/emp")
    public String addEmp(Employee employee) {
        // 来到员工列表页面
        employeeDao.save(employee);
        return "redirect:/emps";
    }    
```



日期格式问题：

There was an unexpected error (type=Bad Request, status=400).

Validation failed for object='employee'. Error count: 1

日期的格式化：SpringMVC将页面提交的值需要转换为指定的类型；

默认日期是按照 / 的方式



### 6）、编辑员工

```html
<!--链接中的变量需要拼接-->
// http://127.0.0.1:8088/ireadygo/emp?id=1001
<a class="btn btn-sm btn-primary" th:href="@{/emp(id=${emp.id})}">编辑</a>

// http://127.0.0.1:8088/ireadygo/emp/1003 : Restful风格
<a class="btn btn-sm btn-danger" th:href="@{/emp/}+${emp.id}">删除</a>

// 三元条件表达式可以忽略 else
<tr th:class="${row.even}? 'alt'">
  ...
</tr>

<!--需要区分员工修改还是添加-->
<form th:action="@{/emp}" method="post">
    <!--发送 put 请求修改员工的数据-->
    <!--
1、SpringMVC中配置HiddenHttpMethodFilter；
2、页面创建一个post表单
3、创建一个input项，name="_method"，值就是我们制定的请求方式
-->
    <input type="hidden" name="_method" th:if="${emp!=null}" value="put">
    <input type="hidden" name="id" 
           th:if="${emp!=null}" th:value="${emp.id}">

    <div class="form-group">
        <label>LastName</label>
        <input name="lastName" type="text" 
               class="form-control" placeholder="zhangsan"
               th:value="${emp!=null}?${emp.lastName}">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input name="email" type="email" 
               class="form-control" placeholder="zhangsan@atguigu.com"
               th:value="${emp!=null}?${emp.email}">
    </div>
    <div class="form-group">
        <label>Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" 
                   name="gender" value="1"
                   th:checked="${emp!=null}?${emp.gender == 1}">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" 
                   name="gender" value="0"
                   th:checked="${emp!=null}?${emp.gender == 0}">
            <label class=" form-check-label">女</label>
        </div>
    </div>
    <div class="form-group">
        <label>department</label>
        <!--提交的是部门的id-->
        <select class="form-control" name="department.id">
            <option th:each="dept : ${depts}" 
                    th:value="${dept.id}" th:text="${dept.departmentName}"
                    th:checked="${emp!=null}?${dept.id}==${emp.department.id}">1
            </option>
        </select>
    </div>
    <div class="form-group">
        <label>Birth</label>
        <input name="birth" type="text" 
               class="form-control" placeholder="zhangsan"
               th:value=
               "${emp!=null}?${#dates.format(emp.birth,'yyy-MM-dd')}">
    </div>
    <button type="submit" 
            class="btn btn-primary" 
            th:text="${emp!=null}?'修改':'添加'"></button>
</form>
```

There was an unexpected error (type=Method Not Allowed, status=405).

Request method 'PUT' not supported



```java
// 来到修改页面，查询当前员工，在页面回显
@GetMapping("/emp/{id}")
public String toEditPage(@PathVariable("id") Integer id, Model model) {
    Employee employee = employeeDao.get(id);
    // 页面显示所有的部门列表
    Collection<Department> departments = departmentDao.getDepartments();
    model.addAttribute("emp", employee);
    model.addAttribute("depts", departments);

    // 回到修改页面（add是一个修改添加二合一的页面）
    return "emp/add";
}

// 员工修改，需要提交员工的id
@PutMapping("/emp")
public String updateEmployee(Employee employee) {
    employeeDao.save(employee);
    return "redirect:/emps";
}
```

### 7）、删除员工

```html
<button class="btn btn-sm btn-danger deleteBtn" 
        th:attr="del_uri=@{/emp/}+${emp.id}">删除</button>
<form id="deleteEmpForm" action="" method="post">
    <!--必须使用 name=_method 否则报405 -->
    <input type="hidden" name="_method" th:value="delete">
</form>

<script>
    $(".deleteBtn").on("click", function () {
        // console.log($(this).attr("del_uri"));
        $("#deleteEmpForm").attr("action", $(this).attr("del_uri")).submit();
    })
</script>
```

```java
// 删除员工
@DeleteMapping("/emp/{id}")
public String deleteEmployee(@PathVariable("id") Integer id) {
    employeeDao.delete(id);
    return "redirect:/emps";
}
```



注意：

使用put或者delete方式提交表单时，input必须添加 name=_method

<!--必须使用 name=_method 否则报405 -->
<input type="hidden" name="_method" th:value="delete">

## 6、错误处理机制

### 1）、SpringBoot默认的错误处理机制

浏览器返回一个默认的错误页面

如果是客户端，返回JSON数据

原理：可以参照 ErrorMvcAutoConfiguration：错误处理的自动配置；

1）如何定制错误响应；

1）如果定制错误页面

2）如何定制错误JSON数据



## 7、配置嵌入式Servlet容器

SpringBoot默认使用的是嵌入的Servlet容器（Tomcat）；

选中pom.xml点击右键，可以查看项目的依赖树

问题？

### 1）、如何定制和修改Servlet容器的相关配置；

方法一、修改和server有关的配置（ServerProperties）

```properties
sever.port=8081
server.servlet.context-path=/ireadygo

// 通用的Servlet容器设置
server.xxx
// Tomcat的设置
server.tomcat.xxx
```

方法二、编写一个ConfigurableServletWebServerFactory：嵌入式的Servlet容器的定制器

```java
// 定义嵌入式的Servlet容器
@Bean
public ConfigurableServletWebServerFactory configurableServletWebServerFactory() {
    TomcatServletWebServerFactory tomcatServletWebServerFactory 
        = new TomcatServletWebServerFactory();
    tomcatServletWebServerFactory.setPort(8888); // 此配置的优先级低于yaml配置文件的优先级
    return tomcatServletWebServerFactory;
}
```

### 2）、注册Servlet三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件

注册三大组件用以下方式

ServletRegistrationBean

```java
// 注册三大组件
@Bean
public ServletRegistrationBean<HttpServlet> servletRegistrationBean() {
    ServletRegistrationBean<HttpServlet> servletRegistrationBean 
        = new ServletRegistrationBean<>(new MyServlet(), "/myServlet");
    return servletRegistrationBean;
}
```

FilterRegistrationBean

```java
@Bean
FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean<Filter> filter = 
        new FilterRegistrationBean<>(new MyFilter());
    filter.setUrlPatterns(Arrays.asList("/hello", "/myServlet"));
    return filter;
}
```

ServletListenerRegistrationBean



SpringBoot帮我们自动配置SpringMVC的时候，**自动的注册SpringMVC的前端控制器：DispatcherServlet；**

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, 
                   name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet) {
    DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet,                                                        this.webMvcProperties.getServlet().getPath());
    // 默认拦截：/ 所有请求；包括静态资源，但是不拦截jsp请求；/*会拦截jsp
    // 可以通过server.servletPath来修改SpringMVC前端控制器默认的拦截路径
    registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
    registration.setLoadOnStartup(this.webMvcProperties.getServlet()
                                  .getLoadOnStartup());
    if (this.multipartConfig != null) {
        registration.setMultipartConfig(this.multipartConfig);
    }
    return registration;
}
```



### 3）、嵌入式Servlet容器自动配置原理

ServletWebServerFactoryAutoConfiguration：嵌入式的Servlet容器自动配置？

```java
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
// 导入了
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
         ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
         ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
         ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {


    @Configuration
    class ServletWebServerFactoryConfiguration {

        @Configuration
        @ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
        @ConditionalOnMissingBean(value = ServletWebServerFactory.class, s
                                  earch = SearchStrategy.CURRENT)
        public static class EmbeddedTomcat {

            // 注册嵌入式Servlet容器工厂
            @Bean
            public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
                return new TomcatServletWebServerFactory();
            }

        }
}
```



TomcatServletWebServerFactory中创建Tomcat；

```java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
    Tomcat tomcat = new Tomcat();
    // 配置Tomcat的基本环境
    File baseDir = (this.baseDirectory != null) ? 
        this.baseDirectory : createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    prepareContext(tomcat.getHost(), initializers);
    return getTomcatWebServer(tomcat);
}

protected TomcatWebServer getTomcatWebServer(Tomcat tomcat) {
    return new TomcatWebServer(tomcat, getPort() >= 0);
}

public TomcatWebServer(Tomcat tomcat, boolean autoStart) {
    Assert.notNull(tomcat, "Tomcat Server must not be null");
    this.tomcat = tomcat;
    this.autoStart = autoStart;
    initialize();
}
```

我们对嵌入式容器的配置修改是怎么生效的？

```java
ServerProperties、ConfigurableServletWebServerFactory 
```



# 五、Docker

## 1、简介

Docker是一个开源的容器 引擎

Docker支持将软件编译成一个镜像；然后在镜像中各种软件做好配置，将镜像发布出去，其它使用者可以直接使用这个镜像；

## 2、核心概念

docker主机（Host）：安装了Docker程序的机器（Docker直接安装在操作系统之上）

docker客户端（Client）：连接docker主机进行操作；

docker仓库（Registry）：用来保存各种打包好的软件镜像；

docker镜像（Images）：





# 六、SpringBoot与数据访问

## 1、JDBC

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/
    		community?serverTimezone=UTC&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
```

默认使用：com.zaxxer.hikari.HikariDataSource 作为数据源

数据源的相关配置都在DataSourceProperties里面；



自动配置原理：

org.springframework.boot.autoconfigure.jdbc

1、参考 DataSourceConfiguration，配置数据源，默认使用HikariDataSource数据源；**可以使用spring.datasource.type 指定自定义的数据源类型**

2、SpringBoot默认可以支持

```、BasicDataSource、HikariDataSource
org.apache.tomcat.jdbc.pool.DataSource
```

3、自定义数据源类型

```java
@Configuration
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        // 使用 DataSourceBuilder创建数据源，利用反射创建相应type的数据源，并且绑定相关属性
        return properties.initializeDataSourceBuilder().build();
    }

}
```

4、

```java
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {
    
@Configuration
@Import({ DataSourceInitializerInvoker.class, DataSourceInitializationConfiguration.Registrar.class })
class DataSourceInitializationConfiguration {

class DataSourceInitializerInvoker implements ApplicationListener<DataSourceSchemaCreatedEvent>, InitializingBean {
    afterPropertiesSet：this.dataSourceInitializer.createSchema();
    onApplicationEvent：initializer.initSchema();
    
DataSourceInitializer
    createSchema（schema脚本）、initSchema（data数据脚本）、runScripts
```

作用：

​	1）createSchema：运行建表语句；

​	2）initSchema：运行插入数据的sql语句；

默认只需要将文件命名为：

```properties
schema-*.sql、data-*.sql
默认规则：schema.sql、schema-all.sql
```

自定义schema脚本位置：

```yaml
initialization-mode: always
    schema:
      - classpath:department.sql
```

5、操作数据库：自动配置了JdbcTemplate操作数据库



## 2、整合Druid数据源

1）配置

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/jdbc?serverTimezone=UTC&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver

    ###################以下为druid增加的配置###########################
    type: com.alibaba.druid.pool.DruidDataSource
    # 下面为连接池的补充设置，应用到上面所有数据源中
    # 初始化大小，最小，最大
    initialSize: 5
    minIdle: 5
    maxActive: 20
    # 配置获取连接等待超时的时间
    maxWait: 60000
    # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    timeBetweenEvictionRunsMillis: 60000
    # 配置一个连接在池中最小生存的时间，单位是毫秒
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    # 打开PSCache，并且指定每个连接上PSCache的大小
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙，此处是filter修改的地方
    filters:
      commons-log.connection-logger-name: stat,wall,log4j
    # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    # 合并多个DruidDataSource的监控数据
    useGlobalDataSourceStat: true
```

配置数据源

```java
@Configuration
public class DruidSource {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource dataSource() {
        return new DruidDataSource();
    }

    // 配置Druid的监控
    // 1、配置一个管理后天的Servlet
    @Bean
    public ServletRegistrationBean StatViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(
            new StatViewServlet(), "/druid/*");
        Map<String, String> initParmas = new HashMap<>();
        initParmas.put("loginUsername", "admin");
        initParmas.put("loginPassword", "123456");
        initParmas.put("allow", "");//默认允许所有访问
        initParmas.put("deny", "192.168.199.128");
        bean.setInitParameters(initParmas);

        return bean;
    }

    // 配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter() {  // 不要写 filterRegistrationBean
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParms = new HashMap<>();
        initParms.put("exclusions", "*.js,*.css,/druid/*");
        bean.setInitParameters(initParms);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return bean;
    }
}
```

## 3、整合MyBatis

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>
```

### 1）、注解版

```java
// 指定这是一个操作数据库的 Mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    Department getDeptById(Integer id);

    @Options(useGeneratedKeys = true, keyProperty = "id") // 使用自增id
    @Insert("insert into department(departmentName) values(#{departmentName})")
    int insertDept(Department department);

    @Delete("delete from department where id = #{id}")
    int deleteDeptById(Integer id);

    @Update("update department set departmentName=#{departmentName} 
            where id = #{id}")
    int updateDept(Department department);
}
```

```java
@GetMapping("/dept")
public Department insertDept(Department department) {
    departmentMapper.insertDept(department);
    return department;  // mapper中必须配置自增id，否则查询不会显示id值
}
```

问题：

自定义MyBatis的配置规则；给容器中添加一个ConfigurationCustomizer

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

批量扫描Mapper配置；

```java
@MapperScan(value = "com.ireadygo.springbootsxt.mapper")
@SpringBootApplication
public class SpringbootsxtApplication {
```

### 2）、配置文件版

```yaml
# 配置 MyBatis
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  // 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml  // 指定sql映射文件的位置
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ireadygo.springbootsxt.mapper.EmployeeMapper">
    <select id="getEmpById" resultType="com.ireadygo.springbootsxt.bean.Employee">
      SELECT * FROM employee WHERE id = #{id}
    </select>

    <insert id="insertEmp">
      INSERT INTO employee(lastName,email,gender,d_id) 
        VALUES (#{lastName},#{email},#{gender},#{dId})
    </insert>
</mapper>
```



## 4、整合 SpringData JPA

### 1）、整合步骤

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
// 配置JPA映射关系
@Entity  // 告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user")
public class User {

    @Id // 这是主键
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 自增主键
    private Integer id;
    @Column(name = "last_name", length = 50) // 这是和数据表对应的一个列
    private String lastName;

    private String email;
    
    // 必须添加get、set方法
```

2）、编写一个dao类操作实体类对应的数据表（Repository）

```java
// 继承JpaRepository来完成对数据的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}
```

3）基本的配置，参考JpaProperties

```yaml
# 配置jpa
spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

# 七、启动配置原理

几个重要的回调机制

配置在META-INF/spring.factories

**ApplicationContextInitializer** 

**SpringApplicationRunListener** 

只需要放在ioc容器中

**ApplicationRunner** 

**CommandLineRunner**



启动流程：

## 1、**创建SpringApplication对象**

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
    	// 保存主配置类
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    	// 判断当前是否是一个web应用
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
    	
    	// 从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；人后保存
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    
    	// 从类路径下找到META-INF/spring.factories配置的所有 ApplicationListener
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    	
    	// 从配置类中找到有 main 方法的主配置类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

![](G:\03_Markdown\Images\Spring启动流程_setInitializers.png)

![](G:\03_Markdown\Images\SpringBoot启动_setListeners.png)

## 2、运行 run 方法

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters 
        = new ArrayList<>();
    configureHeadlessProperty();
    // 获取SpringApplicationRunListeners；从列路径下META-INF/spring.factories
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        // 封装命令行参数
        ApplicationArguments applicationArguments = 
            new DefaultApplicationArguments(args);
        // 创建环境完成后回调SpringApplicationRunListener.environmentPrepared
        ConfigurableEnvironment environment = 
            prepareEnvironment(listeners, applicationArguments);
        
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);

        // 创建ApplicationContext；决定创建web的ioc还是普通的ioc
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);

        // 准备上下文环境，将environment保存到ioc中
  // applyInitializers：回调之前保存的所有的ApplicationContextInitializer的initialize方法
        // 回调所有SpringApplicationRunListener.contextPrepared（）
        prepareContext(context, environment, listeners, 
                       applicationArguments, printedBanner);
        // prepareContext运行完成后调用SpringApplicationRunListeners的contxtLoaded
        
        // ioc容器初始化（如果是web容器还会创建嵌入式的Tomcat）
        // 扫描，创建，加载所有组件的地方
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        // 调用ApplicationRunner和CommandLineRunner的run方法
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```



## 3、事件监听机制

配置在META-INF/spring.factories

**ApplicationContextInitializer** 

**SpringApplicationRunListener** 

只需要放在ioc容器中

**ApplicationRunner** 

**CommandLineRunner**