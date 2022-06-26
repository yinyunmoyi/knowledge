[TOC]



# springboot入门

## springboot与微服务

springboot的特点：

1、可以快速创建一个可以独立运行的spring项目

2、使用嵌入式servlet容器，应用无需打成war包再放入web服务器中，直接可以打成jar包然后用java命令运行

3、starters可以很方便的导入依赖并进行版本控制

4、大量的自动配置，无需xml，开箱即用

5、方便的监控应用的运行情况

微服务的概念：一种架构风格（服务微化），一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；每一个功能元素最终都是一个可独立替换和独立升级的软件单元。

与之对应的是单体应用ALL IN ONE。

微服务与SOA：

SOA是面向服务的架构，它将应用程序的不同功能单元（称为服务）进行拆分，并通过这些服务之间定义良好的接口和协议联系起来。微服务是SOA的升华，两者的区别在于：

1、微服务是去ESB（enterprise service bus，企业服务总线，连接所有需要通信的系统）的，SOA还是以ESB为核心

2、微服务更强调服务的细粒度和重用组合，SOA更强调接口的规范化，对细粒度没有那么高的要求

## 编写helloworld

编写一个功能：浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；

1、创建一个maven工程；（jar）

2、导入spring boot相关的依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

3、编写一个主程序；启动Spring Boot应用

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

4、编写相关的Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

- 默认的包扫描位置：主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来，无需以前的包扫描配置


- 想要改变扫描路径：@SpringBootApplication(scanBasePackages="com.atguigu")

- 或者@ComponentScan 指定扫描路径

5、运行主程序测试

运行后就可以访问localhost:8080/hello就可以看见打印的语句：

![QQ图片20200911205154](D:\笔记\springboot\QQ图片20200911205154.png)

6、以jar包的形式运行：

在pom中加入依赖

```xml
 <!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将这个应用打成jar包，直接使用java -jar 包的形式运行，然后也可以实现上述访问。

## pom文件解析

springboot的父项目是：

~~~xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
</parent>
~~~

点开后发现它的父项目是：

~~~xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-dependencies</artifactId>
		<version>1.5.9.RELEASE</version>
		<relativePath>../../spring-boot-dependencies</relativePath>
	</parent>
~~~

它的一部分如下，这个项目就是定义各种版本的地方：

~~~xml
<properties>
		<!-- Dependency versions -->
		<activemq.version>5.14.5</activemq.version>
		<antlr2.version>2.7.7</antlr2.version>
		<appengine-sdk.version>1.9.59</appengine-sdk.version>
		<artemis.version>1.5.5</artemis.version>
		<aspectj.version>1.8.13</aspectj.version>
		<assertj.version>2.6.0</assertj.version>
		<atomikos.version>3.9.3</atomikos.version>
		<bitronix.version>2.1.4</bitronix.version>
		<caffeine.version>2.3.5</caffeine.version>
		<cassandra-driver.version>3.1.4</cassandra-driver.version>
~~~

pom文件还引入了web相关的启动器：

~~~xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~

在springboot中想使用什么场景就可以引入对应的启动器，可以很方便的开发对应的功能。

## @SpringBootApplication

Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。

观察该注解的定义：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
~~~

### @SpringBootConfiguration

其中有一个注解@SpringBootConfiguration表示它是Spring Boot的配置类；再观察SpringBootConfiguration注解的定义：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
~~~

@Configuration：配置类上来标注这个注解，这个注解就是spring底层的注解，可以用配置类来完成各种bean的初始化，观察这个注解的定义：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    String value() default "";
}
~~~

发现有@Component，代表这个注解也是spring的一个组件。

### @EnableAutoConfiguration

SpringBootApplication注解定义中还有一个注解：@EnableAutoConfiguration，它代表自动开启自动配置功能。观察它的定义：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
~~~

@AutoConfigurationPackage代表自动配置包，在它的定义中：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
~~~

@Import({Registrar.class})是spring底层的注解，代表向容器中导入组件Registrar.class，再看它的定义：

~~~java
@Order(-2147483648)
    static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        Registrar() {
        }

        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
        }

        public Set<Object> determineImports(AnnotationMetadata metadata) {
            return Collections.singleton(new AutoConfigurationPackages.PackageImport(metadata));
        }
    }
~~~

我们将断点打在registerBeanDefinitions方法，然后运行springboot，可以发现以下表达式：

~~~java
(new AutoConfigurationPackages.PackageImport(metadata)).getPackageName()
~~~

的值是：springboot

![QQ图片20200912102358](D:\笔记\springboot\QQ图片20200912102358.png)

而我们的包结构中，启动类所属的包名正是springboot：

![QQ图片20200912102509](D:\笔记\springboot\QQ图片20200912102509.png)

所以@AutoConfigurationPackage会将主配置类所在包的及下面所有子包中的所有组件都扫描到spring容器。

在EnableAutoConfiguration定义中还有一个注解：@Import({EnableAutoConfigurationImportSelector.class})

在这个类的父类中有一个selectImports方法：

~~~java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            try {
                AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
                AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
                List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
                configurations = this.removeDuplicates(configurations);
                configurations = this.sort(configurations, autoConfigurationMetadata);
                Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
                this.checkExcludedClasses(configurations, exclusions);
                configurations.removeAll(exclusions);
                configurations = this.filter(configurations, autoConfigurationMetadata);
                this.fireAutoConfigurationImportEvents(configurations, exclusions);
                return (String[])configurations.toArray(new String[configurations.size()]);
            } catch (IOException var6) {
                throw new IllegalStateException(var6);
            }
        }
    }
~~~

调试这个方法，发现springboot运行时configurations里面装入的就是要自动添加进容器的组件，这些类都是配置类：

![QQ图片20200912105043](D:\笔记\springboot\QQ图片20200912105043.png)

这些配置都是读取自类路径下的META-INF/spring.factories：

![QQ图片20200912105335](D:\笔记\springboot\QQ图片20200912105335.png)

以前我们需要自己配置的东西，自动配置类都帮我们自动完成了。

虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration
按照条件装配规则（@Conditional），最终会按需配置。（有的类没有导入，条件不满足，一些组件就没有注册到容器中）

## controller解析

在之前我们编写的controller类中：

~~~java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
~~~

@ResponseBody代表将数据写给浏览器，如果是对象还能转为json，这个注解也可以放在类上代表所有方法都有这个注解。在类上也可以用@RestController：

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    String value() default "";
}
~~~

此时一个@RestController就代表@Controller、@ResponseBody两个注解。

## 简单配置

在项目的resources下可以设置一个配置文件application.properties：

<img src="D:\笔记\springboot\QQ图片20200912112159.png" alt="QQ图片20200912112159" style="zoom:50%;" />

在其中加入：

~~~
server.port=8081
~~~

再次运行项目就会使开放的端口从默认的8080变为8081.

# springboot配置

SpringBoot使用一个全局的配置文件，配置文件名是固定的：

•application.properties

•application.yml

## yaml

YAML（YAML Ain't Markup Language）是一种配置文件格式，它的特点是以数据为中心。

对于前面我们配置端口的例子，yaml（后缀也可以是yml）配置文件应该这样写：

~~~yaml
server:
  port: 8081
~~~

同样含义的xml写法：

~~~xml
<server>
	<port>8081</port>
</server>
~~~

yaml主要就是用空格来标识格式的，在上述文件中，server作为上一层的属性，而port作为下一层的属性，如果同层还要加入其它属性，只需要让port和新增属性拥有同样的空格缩进数量即可，空格数量可以是1个，也可以是4个或者其他任意个：

~~~yaml
server:
  port: 8081
  path: /hello
~~~

不仅层级用空格来控制，yaml的属性和属性值之间必须用空格来严格区分：

~~~
k:空格v
~~~

字符串默认不用加上单引号或者双引号，单引号和双引号在yaml中的含义不同，单引号中的特殊字符如\n会被翻译为\n，而双引号中的\n会被翻译为回车。

1、对象和键值对的写法：

~~~yaml
friends:
		lastName: zhangsan
		age: 20
~~~

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```

注意行内写法属性名和属性值之间依然隔着空格。

2、数组和set、list：

~~~yaml
pets:
 - cat
 - dog
 - pig
~~~

注意-也要求一个层级缩进值相同，-与值之间隔一个空格

行内写法

```yaml
pets: [cat,dog,pig]
```

行内写法逗号之间不用加空格。

## @ConfigurationProperties

配置文件设置如下：

~~~yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
~~~

这里的lastName也可以用下列属性代替，它们的含义相同：

~~~
last-name: hello
~~~

javaBean：

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能，所以需要加com注解
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    必须要有所有的getset方法
}

public class Dog {
    private String name;
    private Integer age;
	所有的getset方法
}
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

运行测试程序：

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
public class YamlTest {

    @Autowired
    Person person;

    @Test
    public void test() {
        System.out.println(person);
    }
}
~~~

即可将注入person中的属性打印出来。

properties文件一样可以自动注入属性：

~~~properties
person.age=99
person.birth=1017/7/17
person.boss=false
person.dog.name=dog
person.dog.age=15
person.last-name=22
person.lists=a,b,c
person.maps.k1=v1
person.maps.k2=v2
~~~

当两个配置文件同时存在时，properties会生效。

## @Value

value注解也可以将配置文件中的值注入类：

~~~java
@Value("${person.last-name}")
private String lastName;
@Value("#{11*2}")
private Integer age;
@Value("true")
private Boolean boss;
~~~

相比之下，value注解适合注入类的少量属性，而ConfigurationProperties适合专门用bean映射配置文件的场景，两者还有以下区别：

|            | @ConfigurationProperties | @Value |
| ---------- | ------------------------ | ------ |
| 功能         | 批量注入配置文件中的属性             | 一个个指定  |
| 松散绑定（松散语法） | 支持                       | 不支持    |
| SpEL       | 不支持                      | 支持     |
| JSR303数据校验 | 支持                       | 不支持    |
| 复杂类型封装     | 支持                       | 不支持    |

其中松散绑定的意思是在使用@ConfigurationProperties时，配置文件中的属性是以下几种时是等效的：

~~~
person.lastName
person.last-name
person.last_name
PERSON_LAST_NAME
~~~

但是在value注解中如果两者不一致不能取到值。

复杂类型封装的意思是value不支持注入map等复杂类型。

JSR303数据校验是指在类上加入@Validated代表类下字段都可以加入校验，例如下面：

~~~java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    @Email
    private String lastName;
    
    private Integer age;
    
    private Boolean boss;

    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
~~~

@Email注解会检验此字段是否是邮箱，如果不是则校验不通过。

## @PropertySource

可以让springboot加载指定的配置文件，之前的配置都是在全局配置文件中，不方便管理：

~~~java
@PropertySource(value = {"classpath:person.properties"})
@Component
public class Person {
~~~

## 配置类

springboot默认不会引入spring的配置文件，如果想使用必须用@ImportResource导入配置文件，这个注解要放在主配置类上：

~~~java
@ImportResource(locations = {"classpath:beans.xml"})
~~~

springboot更推荐用配置类的方式来代替配置文件，可以自定义一个类并在类上加@Configuration注解，然后用@Bean给容器中添加组件：

~~~java
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
~~~

## 配置文件占位符

在配置文件中可以直接以下列形式赋值随机数：

~~~
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
~~~

还可以获取之前配置过的值：

~~~properties
person.dog.name=${person.hello}_dog
~~~

如果之前的值没有赋值过还可以给出默认值：

~~~properties
person.dog.name=${person.hello:hello}_dog
~~~

## Profile

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

这个profile就是用来设置多个配置文件的关键，比如开发环境和测试环境可以用不同的配置，可以在这两者之间灵活切换。

yaml文件支持多profile：

~~~yaml
server:
  port: 8081
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
  profiles: prod  #指定属于哪个环境
~~~

---代表一个文件有多个分区，最上面的分区指定了生效的profile应该是prod区，然后对应profiles为prod的配置文件区就生效了，其余部分不生效。

我们还可以用命令行参数或者虚拟机参数来指定profile：

~~~
java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
或
-Dspring.profiles.active=dev
~~~

## 配置文件加载位置

springboot 启动会扫描以下位置的主配置文件：

–file:./config/   （项目路径）

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

我们还可以通过启动时加入spring.config.location参数来指定配置文件的位置：

~~~
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
~~~

## 配置文件优先级

SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置：

1.命令行参数

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

多个配置用空格分开； --配置项=值

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值

==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==

**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

==**再来加载不带profile**==

**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**

10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

## 自动配置原理

根据之前的分析，springboot在启动时会向容器中加入一些xxxAutoConfiguration类，以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理：

~~~java
@Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中

@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；    判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
  
  	//他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;
  
   //只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
~~~

根据当前不同的条件判断，决定这个配置类是否生效，condition开头的注解都是用来判断的，一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

所有在配置文件中能配置的属性都是在xxxxProperties类中封装者‘；配置文件能配置什么就可以参照某个功能对应的这个属性类：

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
```

根据上述结果，我们就可以在配置文件中加入spring.http.encoding相关的配置了，这样springboot就能自动设置。

@Conditional是spring原生的注解，当符合某些条件时，才给容器中添加组件，配置类配里面的所有内容才生效：

| @Conditional扩展注解                | 作用（判断是否满足当前指定条件）               |
| ------------------------------- | ------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                |
| @ConditionalOnBean              | 容器中存在指定Bean；                   |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                  |
| @ConditionalOnExpression        | 满足SpEL表达式指定                    |
| @ConditionalOnClass             | 系统中有指定的类                       |
| @ConditionalOnMissingClass      | 系统中没有指定的类                      |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                 |
| @ConditionalOnWebApplication    | 当前是web环境                       |
| @ConditionalOnNotWebApplication | 当前不是web环境                      |
| @ConditionalOnJndi              | JNDI存在指定项                      |

我们可以通过启用  debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效，springboot启动时就可以观察日志：

~~~java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
~~~

自动配置是springboot的精髓，它为我们简化了大量的操作。

## 修改SpringBoot的默认配置的方法总结

模式：

​	1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

​	2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置，继承/实现某个类或接口，按照模式实现相应方法

​	3）、在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置，将某个特定类的对象返回到容器中



总结：

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定


- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了


- 定制化配置


- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。

xxxxxAutoConfiguration ---> 组件  ---> xxxxProperties里面拿值  ----> application.properties

## 使用springboot开发的步骤

- 引入场景依赖


- - <https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter>


- 查看自动配置了哪些（选做）


- - 自己分析，引入场景对应的自动配置一般都生效了
  - 配置文件中debug=true开启自动配置报告。Negative（不生效）\Positive（生效）


- 是否需要修改


- - 参照文档修改配置项


- - - <https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties>
    - 自己分析。xxxxProperties绑定了配置文件的哪些。


- - 自定义加入或者替换组件


- - - @Bean、@Component。。。


- - 自定义器  **XXXXXCustomizer**；
  - ......



## springboot2 ：@Configuration的proxyBeanMethods属性和Full/Lite模式

它的proxyBeanMethods属性默认是true的：

~~~java
@Configuration(proxyBeanMethods = true)
~~~

当配置类的该属性是true的时候，内部互相调用方法得到的都是唯一的对象：

~~~java
@Bean 
public User user01(){
  User zhangsan = new User("zhangsan", 18);
  //user组件依赖了Pet组件
  zhangsan.setPet(tomcatPet());
  return zhangsan;
}

@Bean("tom")
public Pet tomcatPet(){
  return new Pet("tomcat");
}

~~~

比如上述的例子来说，user01实例依赖tomcatPat实例完成注入（多次调用tomcatPat得到的结果唯一）。

直接用编程式的方式获取容器中的实例会发现，注册到容器中的配置类是cglib的代理对象，并不是真正的类，所以直接调用它的方法会得到唯一的结果，保证单实例。

如果该属性改为false，那就不保证单实例了，每次调用都得到不一样的结果。

这就是Full和Lite模式：

Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】

Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】

组件依赖必须使用Full模式默认。其他默认是否Lite模式（Lite模式加载的速度更快）









# 日志

## 日志框架概述

springboot底层用的日志框架是SLF4j和logback。日志框架分两种：日志门面  （日志的抽象层）、日志实现。

日志的抽象层不提供日志的实现，只是提供一个接口，这里的日志门面就是SLF4J，日志实现是Logback；开发时调用日志抽象层的方法即可：

~~~java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
~~~

slf4j和其他日志框架的关系如下：

![concrete-bindings](D:\笔记\springboot\concrete-bindings.png)

从左到右的图：第一个图表示只使用slf4j不使用日志实现无法完成日志的记录；第二个图代表日志抽象层使用slf4j，日志实现层使用logback；第三、四个图代表有些日志实现层与slf4j没有很好的衔接，此时需要一个中间层来适配才能顺利完成日志输出；第五个图代表日志实现层使用slf4j自带的日志实现；第六个图也是无法完成日志打印的。

## 统一替换依赖

在springboot中需要依赖很多框架，其中很多框架的日志打印方式都不一样，但是我们运行springboot时还想要都统一用slf4j来打印，springboot此时是通过下列机制来实现的：

![legacy](D:\笔记\springboot\legacy.png)

左上角的图，最左边的三个方块的关系代表之前我们提到的，slf4j作为日志抽象层，logback作为日志实现层。但是此时项目中还有其他的日志框架，如commons logging、log4j、java.util.logging，此时我们就需要用jcl-over-slf4j包来替换commons logging包（引入新包，排除旧包），这个jcl-over-slf4j就是一个利用适配原理的一个包，它拥有commons logging对外的全部接口，只不过实现变成了调用slf4j，这样就完成了日志框架的统一，同样的道理，其他日志框架的替换也是采用类似的手段。

简单来说使用slf4j分三步：

1、将系统中其他日志框架先排除出去；

2、用中间包来替换原有的日志框架；

3、导入slf4j其他的实现

在springboot中也进行了类似的操作，如Spring框架用的是commons-logging，此时应该将其依赖的日志框架排除：

~~~xml
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<exclusions>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
~~~

在springboot的测试依赖中就有上面部分的设置：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
~~~

## 日志使用

引入slf4j的logger类即可使用日志功能，需要在测试类上加上必要的注解：

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTest {

    Logger logger = LoggerFactory.getLogger(MyTest.class);

    @Test
    public void test() {
        logger.warn("my!");
    }
}
~~~

可以在类路径下引入配置文件logback.xml，可以直接就被日志框架识别：

~~~properties
#日志级别：默认只显示到info
logging.level.com.atguigu=trace

#logging.path=
# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径；
#logging.file=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

#  在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
~~~

| logging.file | logging.path | Example  | Description             |
| ------------ | ------------ | -------- | ----------------------- |
| (none)       | (none)       |          | 只在控制台输出                 |
| 指定文件名        | (none)       | my.log   | 输出日志到my.log文件           |
| (none)       | 指定目录         | /var/log | 输出到指定目录的 spring.log 文件中 |

各个日志框架使用的配置文件名：

| Logging System          | Customization                            |
| ----------------------- | ---------------------------------------- |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`      |
| JDK (Java Util Logging) | `logging.properties`                     |

可以使用原生的配置文件，也可以用带spring后缀的：如logback-spring.xml，日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能：

~~~xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <!--在dev开发环境生效该配置，可以指定对应的profile控制 -->
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <!--在非dev开发环境生效该配置 -->
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
~~~

# web开发

在springboot开发中，可以在idea中new project - spring initializer，选择对应的模块，就可以自动引入对应的依赖。

## 静态资源的映射规则

关于web开发相关的自动配置，有两个类很关键，一个是ResourceProperties，它可以设置和静态资源有关的参数，缓存时间等；一个是WebMvcAuotConfiguration，它配置了静态资源的文件夹和欢迎页面、自定义图标（在浏览器访问时出现的小图标）

springboot可以采用webjars的方式引入静态资源，如js，在以下网址就可以找到jquery的依赖：

http://www.webjars.org/

~~~xml
<!--引入jquery-webjar-->在访问的时候只需要写webjars下面资源的名称即可
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>jquery</artifactId>
			<version>3.3.1</version>
		</dependency>
~~~

引入项目后，其目录结构：

![搜狗截图20180203181751](D:\笔记\springboot\搜狗截图20180203181751.png)

可以在浏览器上直接访问localhost:8080/webjars/jquery/3.3.1/jquery.js即可访问到。

其他访问静态资源的方式，有以下几个文件夹，可以在浏览器下直接访问文件，springboot会自动去这些文件夹中寻找：

~~~
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
~~~

欢迎文件应该被命名为index.html，此时直接访问localhost:8080/即可访问到。

自定义图标应该被命名为favicon.ico，这两个文件都应该放在静态资源的文件夹路径下。

也可以在配置文件中自定义静态资源文件夹：

~~~
spring.resources.static-locations=classpath:hello/
~~~

## 静态资源的配置原理

- SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）
- SpringMVC功能的自动配置类 WebMvcAutoConfiguration，生效

```
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

- 给容器中配了什么。

```
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
```

- 配置文件的相关属性和xxx进行了绑定。WebMvcProperties==**spring.mvc**、ResourceProperties==**spring.resources**

#### 1、配置类只有一个有参构造器

```
	//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
	public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
```

#### 2、资源处理的默认规则

```
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			//webjars的规则
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```

```
spring:
#  mvc:
#    static-path-pattern: /res/**

  resources:
    add-mappings: false   禁用所有静态资源规则
```

```
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

#### 3、欢迎页的处理规则

```
	HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            //要用欢迎页功能，必须是/**
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}

```

#### 

## 请求参数处理

- Rest风格支持（使用HTTP请求方式动词来表示对资源的操作）


- - 以前：/getUser   获取用户     /deleteUser 删除用户    /editUser  修改用户       /saveUser 保存用户
  - 现在： /user    GET-获取用户    DELETE-删除用户     PUT-修改用户      POST-保存用户

~~~java
   @RequestMapping(value = "/user",method = RequestMethod.GET)  // 也可以用@GetMapping，下同
    public String getUser(){
        return "GET-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String saveUser(){
        return "POST-张三";
    }


    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public String putUser(){
        return "PUT-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public String deleteUser(){
        return "DELETE-张三";
    }
~~~

在后台对各种类型的http请求做好处理之后就面临一个问题，前台的表单提交只有get和post两种，如何能映射到后台的put和delete方法呢，做法是在前台表单method=post，隐藏域 _method=put：

![QQ图片20210912232801](D:\笔记\springboot\QQ图片20210912232801.png)

再开启配置：

~~~yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
~~~

就可以完成上述功能。（用postman等工具直接访问的话就没有这种转化的需求）

原理：

- 请求过来被HiddenHttpMethodFilter拦截


- 请求是否正常，并且是POST


- 获取到_method的值。

  兼容以下请求；PUT.DELETE.PATCH


- 原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。

  过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。


根据这样的原理，只要定制一个自己的HiddenHttpMethodFilter，就可以不用_method来标记，而是用自己自定义的值：

![QQ图片20210918230806](D:\笔记\springboot\QQ图片20210918230806.png)

## 请求映射原理

springboot的底层是springmvc，所有请求的功能入口都是org.springframework.web.servlet.DispatcherServlet，它的父类重写了servlet的doPost，最后调用到doDispath方法：

~~~java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
~~~

处理器映射会找到能够处理对应请求的处理器，一般项目中有以下几个处理器：

![image](D:\笔记\springboot\image.png)

其中**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则。

![image (1)](D:\笔记\springboot\image (1).png)

所有的请求映射都在HandlerMapping中。

- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping


- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。


- - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping


- 我们需要一些自定义的映射处理，我们也可以自己给容器中放HandlerMapping。自定义 HandlerMapping

~~~java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
      for (HandlerMapping mapping : this.handlerMappings) {
        HandlerExecutionChain handler = mapping.getHandler(request);
        if (handler != null) {
          return handler;
        }
      }
    }
    return null;
  }
~~~

## 常用参数注解

@PathVariable（获取路径变量，如car/3/owner/lisi，获取其中的值，如3或者lisi）、@RequestHeader（获取请求头中的参数）、@ResquestAttribute（获取httprequest中的attribute）、@RequestParam（获取请求参数，如car/name=lisi中的lisi）、@MatrixVariable（获取矩阵变量，类似请求参数的一种传递信息的方式）、@CookieValue（获取cookie的值）、@RequestBody（获取请求体中的值）

上面这些注解都是可以取单个值，也可以包装成一个map或者其他对象获取所有值的：

~~~java
@RestController
public class ParameterTestController {


    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv,
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header,
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params,
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){


        Map<String,Object> map = new HashMap<>();

//        map.put("id",id);
//        map.put("name",name);
//        map.put("pv",pv);
//        map.put("userAgent",userAgent);
//        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }


    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }


    //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd
    //2、SpringBoot默认是禁用了矩阵变量的功能，需要开启，开启方式就是自定义一个UrlPathHelper
    //      手动开启：原理。对于路径的处理。UrlPathHelper进行解析。
    //              removeSemicolonContent（移除分号内容）支持矩阵变量的
    //3、矩阵变量必须有url路径变量才能被解析
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10

    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;

    }

}
~~~

## servlet API

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId这些都可以当做方法参数传入控制器中，springmvc会帮我们把它们封装好

**ServletRequestMethodArgumentResolver  **记载着上述参数：

~~~java
@Override
	public boolean supportsParameter(MethodParameter parameter) {
		Class<?> paramType = parameter.getParameterType();
		return (WebRequest.class.isAssignableFrom(paramType) ||
				ServletRequest.class.isAssignableFrom(paramType) ||
				MultipartRequest.class.isAssignableFrom(paramType) ||
				HttpSession.class.isAssignableFrom(paramType) ||
				(pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
				Principal.class.isAssignableFrom(paramType) ||
				InputStream.class.isAssignableFrom(paramType) ||
				Reader.class.isAssignableFrom(paramType) ||
				HttpMethod.class == paramType ||
				Locale.class == paramType ||
				TimeZone.class == paramType ||
				ZoneId.class == paramType);
	}
~~~

## 复杂参数和自定义对象参数

控制器还可以传入下列参数：**Map**、**Model（map、model里面的数据会被放在request的请求域  request.setAttribute）、**Errors/BindingResult、**RedirectAttributes（ 重定向携带数据）**、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

操作这些参数，就相当于操作request，下一次重定向时还可以将这些信息取出

除此之外控制器还支持传入自定义参数，如下列表单提交：(@Data是lombok)

~~~java
/**
 *     姓名： <input name="userName"/> <br/>
 *     年龄： <input name="age"/> <br/>
 *     生日： <input name="birth"/> <br/>
 *     宠物姓名：<input name="pet.name"/><br/>
 *     宠物年龄：<input name="pet.age"/>
 */
@Data
public class Person {
    
    private String userName;
    private Integer age;
    private Date birth;
    private Pet pet;
    
}

@Data
public class Pet {

    private String name;
    private String age;

}
~~~

可以给控制器直接传入Person和Pet参数，springmvc可以自动解析。

当传入参数比较复杂时，还可以定义解析规则，如要实现下列需求，表单提交一个pet参数，参数值的前半部分被解析为name，后半部解析为age：

![QQ图片20210919223501](D:\笔记\springboot\QQ图片20210919223501.png)

自定义一个WebMvcConfigurer，重写addFormatters方法，完成自定义的转化逻辑：

![QQ图片20210919223333](D:\笔记\springboot\QQ图片20210919223333.png)

## 参数处理原理

略

## 数据响应

给Controller中的方法加入ResponseBody注解，然后返回一个对象，前台即可收到这个对象的json信息。

要完成数据响应要引入springboot的web-starter，它引入了json相关的依赖：

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
web场景自动引入了json场景
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>
~~~

阅读源码可以得到springmvc可以返回的所有类型：

~~~
ModelAndView
Model
View
ResponseEntity 
ResponseBodyEmitter
StreamingResponseBody
HttpEntity
HttpHeaders
Callable
DeferredResult
ListenableFuture
CompletionStage
WebAsyncTask
有 @ModelAttribute 且为对象类型的
方法有@ResponseBody 注解 ---> RequestResponseBodyMethodProcessor处理；
~~~

底层解析这些返回值的是返回值解析器，下面是springboot默认的所有返回值解析器：

![image (2)](D:\笔记\springboot\image (2).png)

返回值解析器有两个关键的方法：返回支持处理的类型、处理返回值：

![image (3)](D:\笔记\springboot\image (3).png)

解析的过程：

- 1、返回值处理器判断是否支持这种类型返回值 supportsReturnType
- 2、返回值处理器调用 handleReturnValue 进行处理


- 3、RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的。


- - 1. 利用 MessageConverters 进行处理 将数据写为json


- - - 1、内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型），如下，请求头中的accept字段（q是优先级）：

      ![image (4)](D:\笔记\springboot\image (4).png)

    - 2、服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据。和前者对比最后得到结果：类型转化前的结果、类型转化后的结果


- - - 3、SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？


- - - - 1、得到MappingJackson2HttpMessageConverter可以将对象写为json
      - 2、利用MappingJackson2HttpMessageConverter将对象转为json再写到response。

HTTPMessageConverter消息转化器的方法，关键是能读取什么类型，能转化写为什么类型：

![image (5)](D:\笔记\springboot\image (5).png)

HttpMessageConverter: 看是否支持将 此 Class类型的对象，转为MediaType类型的数据。

例子：Person对象转为JSON。或者 JSON转为Person

springmvc有默认的下列消息转化器：

![image (6)](D:\笔记\springboot\image (6).png)

（

这些convert都是如何初始化出来的？有些是默认new出来装进去的，有些需要导入相关依赖才会装入，装入的动作在WebMvcConfigurationSupport类中，判断是否有依赖导入是通过代码：

jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);

）

观察上述消息转换器，它们能读取的类型如下：

1 - String

2 - String

3 - Resource

4 - ResourceRegion

5 - DOMSource.**class \ **SAXSource.**class**) \ StAXSource.**class \**StreamSource.**class \**Source.**class**

**6 - **MultiValueMap

7 - **true **

**8 - true**

9 - 支持注解方式xml处理的。

（遍历到第7和第8个总是返回true，它们什么类型都能转化）

具体到ResponseBody注解返回一个对象这个例子，最后MappingJackson2HttpMessageConverter  把对象转为JSON（利用底层的jackson的objectMapper转换的），然后写入response返回出去：

![image (7)](D:\笔记\springboot\image (7).png)

## 内容协商

内容协商就是根据客户端接收能力不同，返回不同媒体类型的数据。

首先引入xml的依赖：

~~~java
 <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
~~~

比如用postman发送请求，变换不同的accept，请求返回值的格式就不同（application/xml、application/json）：

![image (8)](D:\笔记\springboot\image (8).png)

浏览器发送请求的accept一般无法受控制，也就无法修改请求中的accept，这个时候可以通过请求参数来完成不同的内容协商，首先要在配置中开启内容协商：

~~~yaml
spring:
    contentnegotiation:
      favor-parameter: true  #开启请求参数内容协商模式
~~~

然后在发送请求时加上format就能完成内容协商：

 <http://localhost:8080/test/person?format=json>

[http://localhost:8080/test/person?format=](http://localhost:8080/test/person?format=json)xml

###内容协商的过程

1、判断当前响应头中是否已经有确定的媒体类型。MediaType

2、获取客户端（PostMan、浏览器）支持接收的内容类型。（获取客户端Accept请求头字段）【application/xml】

- - contentNegotiationManager 内容协商管理器 默认使用基于请求头的策略（默认策略就这一个）：

    HeaderContentNegotiationStrategy  确定客户端可以接收的内容类型

    ![image (9)](D:\笔记\springboot\image (9).png)





解析完毕后，请求头中的accept就被解析到其中：

![image (10)](D:\笔记\springboot\image (10).png)

3、遍历循环所有当前系统的 **MessageConverter**，看谁支持操作这个对象（Person），也就是谁支持读

4、找到支持操作Person的converter，把converter支持的媒体类型统计出来。相当于统计可以写为什么类型

5、统计结果：客户端需要【application/xml】。服务端能力【10种、json、xml】：

![image (11)](D:\笔记\springboot\image (11).png)

6、进行内容协商的最佳匹配媒体类型

7、用 支持 将对象转为 最佳匹配媒体类型 的converter。调用它进行转化 。

### 自定义MessageConverter

为了实现多协议数据兼容，比如accept的值是一个自定义的值：x-guigu，按照我们自定义的格式完成数据响应。

此时需要在WebMvcConfigurer添加我们自定义的MessageConverter：

~~~java
public class GuiguMessageConverter implements HttpMessageConverter<Person> {
    @Override
    public boolean canRead(Class<?> aClass, @Nullable MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Class<?> aClass, @Nullable MediaType mediaType) {
        return aClass.isAssignableFrom(Person.class);
    }

    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-guigu");
    }

    @Override
    public Person read(Class<? extends Person> aClass, HttpInputMessage httpInputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override
    public void write(Person person, @Nullable MediaType mediaType, HttpOutputMessage httpOutputMessage) throws IOException, HttpMessageNotWritableException {
      // 自定义数据转化
        String data = person.getName() + ":" + person.getAge();
        httpOutputMessage.getBody().write(data.getBytes());
    }
}
~~~

~~~java
 @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {

            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
				converters.add(new GuiguMessageConverter());
            }
        }
    }
~~~

这样，方法返回Person类型的对象时，就会按照我们自定义的逻辑，把数据放入response中。

我们还可以写自定义的策略类，完成基于请求参数的自定义内容协商，如访问：

[http://localhost:8080/test/person?format=](http://localhost:8080/test/person?format=json)guigu

代表一种自定义的内容协商格式，但是要注意会覆盖默认的策略，要注意与已经有的策略类兼容，将原来的默认策略类也放到策略的list中。


## thymeleaf

### 模板引擎

jsp和thymeleaf都是模板引擎，它的作用如下图：

![template-engine](D:\笔记\springboot\template-engine.png)

将模板和数据结合起来生成html，完成与前端的交互。在springboot中无法直接使用jsp，它推荐使用模板引擎thymeleaf。

### 引入thymeleaf

加入依赖：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
          	2.1.6
		</dependency>
~~~

在父项目中要调整版本号，使其兼容：

~~~xml
<properties>
		<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
		<!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
		<!-- thymeleaf2   layout1-->
		<thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
  </properties>
~~~

### thymeleaf简单使用

ThymeleafProperties封装了thymeleaf的一些属性，如前缀、后缀：

~~~java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
  	//
~~~

只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染，如把一个文件放入以下位置：

classpath:/templates/success.html，然后运行springboot程序：

~~~java
@Controller
public class HelloController {

    @RequestMapping("/success")
    public String hello(){
        return "success";
    }
}
~~~

在浏览器中访问localhost:8080/success即可通过thymeleaf的方式访问到对应文件。

在对应的模板路径放入html文件：

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
~~~

第二行是thymeleaf的名称空间，用于提示。th：text是设置文本内容，其中的内容是从后台程序传过来的：

~~~java
@Controller
public class HelloController {

    @RequestMapping("/success")
    public String hello(Map<String, Object> map){
    	map.put("hello", "value");
        return "success";
    }
}
~~~

这样设置之后，页面就会显示value（而不会显示“这是显示欢迎信息”，只有直接以普通方式访问html才会显示）

### 语法规则

在thymeleaf中可以用th：来替换任意html属性，方便的进行数据传递，更多的详细功能如下，和jsp一样，也拥有条件判断、循环遍历的功能：

![2018-02-04_123955](D:\笔记\springboot\2018-02-04_123955.png)

表达式、内置对象和工具：

~~~properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；OGNL；
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象：
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
                
                ${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
   <div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
    </div>
    
    Message Expressions: #{...}：获取国际化内容
    Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
    Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">...</div>
    		
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
~~~

## 视图解析原理

方法返回的过程中：

1、目标方法处理的过程中，所有数据都会被放在 **ModelAndViewContainer 里面。包括数据和视图地址**

2、如果方法的参数是一个自定义类型对象（从请求参数中确定的），也会把它重新放ModelAndViewContainer 

3、任何目标方法执行完成以后都会返回、封装成 ModelAndView（数据和视图地址）。

4、processDispatchResult  处理派发结果（页面改如何响应）**

- 1、**render**(**mv**, request, response); 该方法进行页面渲染逻辑


- - 1、根据方法的String返回值得到 **View **对象【它定义了页面的渲染逻辑】


- - - 1、所有的视图解析器尝试是否能根据当前返回值得到**View**对象
    - 2、如果返回值是  redirect:/main.html  --> 最后就会生成这个view对象 Thymeleaf new RedirectView()


- - - 3、ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。
    - 4、view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作


- - - - **RedirectView 如何渲染【重定向到一个页面】**
      - 1、获取目标url地址


- - - - 2、最后调用servlet的原生API，response.sendRedirect(encodedURL);

以Thymeleaf 为例进行视图解析：

- - 返回值以 forward: 开始： new InternalResourceView(forwardUrl); -->  转发request.getRequestDispatcher(path).forward(request, response); 
  - 返回值以 redirect: 开始： new RedirectView() --》 render就是重定向 


- - 返回值是普通字符串： new ThymeleafView（）---> 

## 拦截器

拦截器可以在请求到对应方法前进行判断，如果不满足要求则拦截请求不执行，常用于校验登录动作。

首先需要自定义一个拦截器类实现HandlerInterceptor 接口，实现它的三个方法：

~~~java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

    /**
     * 目标方法执行之前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

    /**
     * 目标方法执行完成以后
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    /**
     * 页面渲染以后
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
~~~

然后通过addInterceptors，将自定义的拦截器配置好：

~~~java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
~~~

拦截器执行的过程：

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

2、先来**顺序执行 **所有拦截器的 preHandle方法

- 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发 **afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion

![image (12)](D:\笔记\springboot\image (12).png)

## 文件上传

页面表单：

~~~html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="submit" value="提交">
</form>
~~~

文件上传代码：

springboot会帮我们把文件封装成MultipartFile类，调用它的transferTo方法就会把文件写到本地

~~~java
    /**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }
        return "main";
    }
~~~

文件上传的配置都在：自动配置类-MultipartAutoConfiguration-MultipartProperties（文件默认是有上传大小限制的，默认是1MB，可以修改这个参数）

文件上传的过程：

- springboot自动配置好了 **StandardServletMultipartResolver   【文件上传解析器】**
- **原理步骤**


- - **1、请求进来使用文件上传解析器判断（**isMultipart**）并封装（**resolveMultipart，**返回**MultipartHttpServletRequest**）文件上传请求**
  - **2、参数解析器来解析请求中的文件内容封装成MultipartFile**


- - **3、将request中文件信息封装为一个Map；**MultiValueMap<String, MultipartFile>  方法中的MultipartFile类型的入参实际上就是根据名称从这个map中取到对象

中间通过FileCopyUtils来实现文件流的拷贝







## springmvc

### 自动配置

SpringBoot对SpringMVC的默认配置都是通过WebMvcAutoConfiguration实现的，自动配置了如下内容：

1、自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），决定了页面跳转、转发与重定向、前后缀等），包括ContentNegotiatingViewResolver、BeanNameViewResolver

2、静态资源文件夹、静态首页访问index.html、图标favicon.ico

3、自动注册了转换器Converter、格式化器Formatter、MessageCodesResolver (see below).定义错误代码生成规则。

自定义上述内容只需将自己的组件注册在容器中即可。

### 扩展springmvc

编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc，如：

~~~java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /atguigu 请求来到 success
        registry.addViewController("/atguigu").setViewName("success");
    }
}
~~~

这样就可以向springmvc添加组件了。

原理在于，在SpringMVC的自动配置类WebMvcAutoConfiguration中有以下注解：

@Import(EnableWebMvcConfiguration.class)

在EnableWebMvcConfiguration中有一部分专门把容器中所有的WebMvcConfigurer都注册：

~~~java
    @Configuration
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
      private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	 //从容器中获取所有的WebMvcConfigurer
      @Autowired(required = false)
      public void setConfigurers(List<WebMvcConfigurer> configurers) {
          if (!CollectionUtils.isEmpty(configurers)) {
              this.configurers.addWebMvcConfigurers(configurers);
            	//一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
            	@Override
             // public void addViewControllers(ViewControllerRegistry registry) {
              //    for (WebMvcConfigurer delegate : this.delegates) {
               //       delegate.addViewControllers(registry);
               //   }
              }
          }
	}
~~~

这样的效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

### 全面接管springmvc

在上述配置类中加上@EnableWebMvc即可全面接管springmvc，也就是自动配置全部失效，连静态资源文件夹都失效了。

原理，在@EnableWebMvc中导入了DelegatingWebMvcConfiguration类：

~~~java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
~~~

而DelegatingWebMvcConfiguration是一个WebMvcConfigurationSupport类

~~~java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
~~~

在springmvc的自动配置类WebMvcAutoConfiguration中：

~~~java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
~~~

因为容器中有了WebMvcConfigurationSupport，所以自动配置不生效。导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能。

## RestfulCRUD

### 默认访问首页、引入bootstrap、默认访问名

访问首页时要把首页index.html放在模板加载路径下，这样才能通过模板引擎的渲染，但是如果直接访问根路径会默认到静态资源文件夹下去寻找index.html，无法直接访问模板加载路径下的index.html，此时应该在controller中加入下列方法，通过这一层转换就会默认去模板路径下寻找index.html了：

~~~java
@RequestMapping({"/", "/index.html"})
public String index() {
    return "index";
}
~~~

也可以在我们的配置类中加入自定义的视图控制器：（不要完全接管springmvc），注意要加@Bean将组件注册在容器：

~~~java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
//@EnableWebMvc   不要接管SpringMVC
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /atguigu 请求来到 success
        registry.addViewController("/atguigu").setViewName("success");
    }

    //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
            }
        };
        return adapter;
    }
}
~~~

下面这两行代码就是注册视图控制器：

~~~java
registry.addViewController("/").setViewName("login");
registry.addViewController("/index.html").setViewName("login");
~~~

相当于：

~~~java
@RequestMapping({"/", "/index.html"})
public String index() {
    return "login";
}
~~~

我们在项目中以webjars的方式引入bootstrap的依赖，然后在html中引入样式文件时，用模板的功能替换掉原有的属性，转而使用bootstrap的样式。

最后还需要在配置文件中定义一个项目访问的默认访问名：

~~~properties
server.context-path=/crud
~~~

这样访问该项目所有资源时都需要加这个前缀。

### 国际化（跳过）

### 登录

在controller中添加：

~~~java
@PostMapping(value = "user/login")
public String login(@RequestParam("username") String username, 
                   @RequestParam("password") String password,
                   Map<String, Object> map) {
    if (!StringUtils.isEmpty(username) && "123456".equals(password)) {
        //登录成功，跳转到登录后
        return "dashboard";
    } else {
        // 给前台设置错误信息
        map.put("msg", "密码错误");
        return "login";
    }
}
~~~

@PostMapping(value = "user/login")注解相当于以下注解，用来接收post请求：

~~~java
@RequestMapping(value = "user/login", method = RequestMethod.POST)
~~~

同样的类似的注解还有：

~~~java
@DeleteMapping
@PutMapping
@GetMapping
~~~

开发期间模板引擎页面修改以后，要实时生效便于调试，这时需要：

1、禁用模板引擎的缓存：

~~~properties
# 禁用缓存
spring.thymeleaf.cache=false 
~~~

2、页面修改完成以后ctrl+f9：重新编译；

页面的简单跳转不能实现登录的功能，因为此时如果登录后跳转完成再刷新会出现表单重复提交的情况，这里的跳转方式应该是重定向，而且在登录成功后应该向session中添加登录凭证，在登录方法参数中应该加入HttpSession参数：

~~~java
session.setAttribute("loginUser", username);
return "redirect:/main.html";
~~~

此外应该禁止用户在未登录的状态下访问其他页面，此时我们需要自定义一个拦截器，在方法访问前先检查session中的登录凭证，如果没有就禁止登录：

~~~java

/**
 * 登陆检查，
 */
public class LoginHandlerInterceptor implements HandlerInterceptor {
    //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if(user == null){
            //未登陆，返回登陆页面
            request.setAttribute("msg","没有权限请先登陆");
            request.getRequestDispatcher("/index.html").forward(request,response);
            return false;
        }else{
            //已登陆，放行请求
            return true;
        }

    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

~~~

在我们的自定义springmvc配置类WebMvcConfigurerAdapter中注册这个拦截器：

~~~java
//所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
                registry.addViewController("/main.html").setViewName("dashboard");
            }

            //注册拦截器
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                //super.addInterceptors(registry);
                //静态资源；  *.css , *.js
                //SpringBoot已经做好了静态资源映射
                registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                        .excludePathPatterns("/index.html","/","/user/login");
            }
        };
        return adapter;
    }
~~~

这里addInterceptors中注册了我们自定义的拦截器，同时指定了拦截除了三个路径的所有其他路径，这三个分别对应两个登录页面和一个登录请求。

### 架构设计

1）、RestfulCRUD：CRUD满足Rest风格；

URI：  /资源名称/资源标识       HTTP请求方式区分对资源CRUD操作

|      | 普通CRUD（uri来区分操作）        | RestfulCRUD       |
| ---- | ----------------------- | ----------------- |
| 查询   | getEmp                  | emp---GET         |
| 添加   | addEmp?xxx              | emp---POST        |
| 修改   | updateEmp?id=xxx&xxx=xx | emp/{id}---PUT    |
| 删除   | deleteEmp?id=1          | emp/{id}---DELETE |

2）、实验的请求架构：

| 实验功能               | 请求URI | 请求方式   |
| ------------------ | ----- | ------ |
| 查询所有员工             | emps  | GET    |
| 查询某个员工(来到修改页面)     | emp/1 | GET    |
| 来到添加页面             | emp   | GET    |
| 添加员工               | emp   | POST   |
| 来到修改页面（查出员工进行信息回显） | emp/1 | GET    |
| 修改员工               | emp   | PUT    |
| 删除员工               | emp/1 | DELETE |

在开发中需要做日期的格式化：SpringMVC将页面提交的值需要转换为指定的类型，此时需要在配置文件中加入可以接受的日期格式：

~~~properties
spring-mvc-date-format=yyyy-MM-dd
~~~







# 错误处理机制

## springboot的默认处理机制

当在浏览器上访问服务器不存在的页面时会出现：

![搜狗截图20180226173408](D:\笔记\springboot\搜狗截图20180226173408.png)

而在postman中模拟http请求访问服务器不存在的资源时会返回一个json：

![搜狗截图20180226173527](D:\笔记\springboot\搜狗截图20180226173527.png)

之所以会出现不同的响应是因为仔细观察两者发出的http请求，浏览器发出的请求中的请求头属性中表示接受的返回格式是html：

~~~
Accept:text/html
~~~

而postman中接受所有消息：

~~~
Accept:*/*
~~~

## 错误处理原理

错误处理的自动配置类是ErrorMvcAutoConfiguration

错误处理的过程大致如下：一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被BasicErrorController处理；

1、在ErrorPageCustomizer中，指定了出现错误后来到error请求进行处理：

~~~java
	@Value("${error.path:/error}")
	private String path = "/error";  系统出现错误以后来到error请求进行处理；（web.xml注册的错误页面规则）
~~~

2、BasicErrorController负责处理默认/error请求：分别浏览器端和非浏览器客户端的请求

~~~java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    
    @RequestMapping(produces = "text/html")//产生html类型的数据；浏览器发送的请求来到这个方法处理
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

	@RequestMapping
	@ResponseBody    //产生json数据，其他客户端来到这个方法处理；
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
~~~

3、DefaultErrorViewResolver解析相应页面，会尝试用模板引擎解析，如果不想再在静态资源文件夹下找error文件夹：

~~~java
@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
			Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot可以去找到一个页面？  error/404
		String errorViewName = "error/" + viewName;
        
        //模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
				.getProvider(errorViewName, this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        //模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面   error/404.html
		return resolveResource(errorViewName, model);
	}
~~~

4、DefaultErrorAttributes在错误处理的结果中添加了很多的属性，在定制错误页面时都可以取用这些属性：

~~~java
帮我们在页面共享信息；
@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, requestAttributes);
		addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
		addPath(errorAttributes, requestAttributes);
		return errorAttributes;
	}
~~~

## 自定义错误处理机制

### 定制错误页面

1、有模板引擎的情况下去error/状态码访问; 【将错误页面命名为  错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到对应的页面，如error/404.html。我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）

页面能获取的信息：

~~~
timestamp：时间戳
status：状态码
error：错误提示
exception：异常对象
message：异常消息
errors：JSR303数据校验的错误都在这里
~~~

2、没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找，还是相同的路径与文件命名方式

3、以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

### 定制错误的json数据

我们可以自定义一个异常处理器来处理我们自己封装的异常UserNotExistException：

~~~java
@ControllerAdvice
public class MyExceptionHandler {

    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}
~~~

但是这样设置之后，任何种类的请求都只能返回json，不能返回错误页面了，所以我们需要在这里转发到/error请求，走springboot的默认错误处理机制，然后根据赋值的错误码来到自定义的页面：

~~~java
@ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
         * Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        //转发到/error
        return "forward:/error";
    }
~~~

### 设置错误响应的数据

出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes方法得到的（是AbstractErrorController（ErrorController）规定的方法）；我们要想加数据只需编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】：

~~~java
//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company","atguigu");
        return map;
    }
}
~~~

加上我们在异常处理器中封装的数据，最后的错误json相应如下：

![搜狗截图20180228135513](D:\笔记\springboot\搜狗截图20180228135513.png)

# Servlet容器

SpringBoot默认使用Tomcat作为嵌入式的Servlet容器：

![搜狗截图20180301142915](D:\笔记\springboot\搜狗截图20180301142915.png)

## servlet相关配置

springboot用来管理servlet相关配置的类是ServerProperties，修改相关配置的方法有两种：

1、直接在配置文件修改：

~~~properties
server.port=8081
server.context-path=/crud

server.tomcat.uri-encoding=UTF-8
~~~

最后配置会加载到ServerProperties类

2、将相关配置加入spring容器：

~~~java
@Bean  //一定要将这个定制器加入到容器中
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {

        //定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}
~~~

ServerProperties类本质上也是EmbeddedServletContainerCustomizer

## 注册servlet三大组件

### 使用配置类RegistrationBean

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件，springboot通过配置类的方式实现：

1、servlet的注册：

注册自定义的servlet类MyServlet，并指定对应的映射路径：

~~~java
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
    return registrationBean;
}
~~~

2、Filter的注册：

注册自定义的MyFilter，并指定对应的过滤路径：

~~~java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
~~~

3、Listener的注册：

~~~java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
~~~

### 使用Servlet API

推荐用这种方式注册servlet，那就是自定义好servlet、Filter、Listener类后，在类上分别加上下列注解，也可以替代配置文件：

~~~java
@ServletComponentScan(basePackages = "com.atguigu.admin") :指定原生Servlet组件都放在那里
@WebServlet(urlPatterns = "/my")：效果：直接响应，没有经过Spring的拦截器？
@WebFilter(urlPatterns={"/css/*","/images/*"})
~~~

这三个注解都是Servlet3的注解，要想让它们生效还需要在springboot的启动类上加上扫描注解：

~~~java
@ServletComponentScan(basePackages = "com.atguigu.admin") :指定原生Servlet组件都放在那里
~~~

### 原生和非原生的servlet

非原生的servlet：SpringBoot帮我们使用SpringMVC的时候，自动的注册SpringMVC的前端控制器，在DispatcherServletAutoConfiguration中会取到servlet的拦截设置然后生效（默认映射的是 / 路径）：

~~~java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
      DispatcherServlet dispatcherServlet) {
   ServletRegistrationBean registration = new ServletRegistrationBean(
         dispatcherServlet, this.serverProperties.getServletMapping());
    //默认拦截： /  所有请求；包静态资源，但是不拦截jsp请求；   /*会拦截jsp
    //可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
    
   registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
   registration.setLoadOnStartup(
         this.webMvcProperties.getServlet().getLoadOnStartup());
   if (this.multipartConfig != null) {
      registration.setMultipartConfig(this.multipartConfig);
   }
   return registration;
}
~~~

当配置了原生servlet之后，如果是相同路径谁优先生效？

根据servlet的精确优选原则，相同路径下，匹配更精确的优先，如 /my/和/my/1比后者优先









## 替换为其他嵌入式Servlet容器

从类结构中我们可以发现springboot还支持其他的嵌入式容器：

![搜狗截图20180302114401](D:\笔记\springboot\搜狗截图20180302114401.png)

springboot默认的servlet容器是tomcat：

~~~xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器；
</dependency>
~~~

还可以使用Jetty，需要先排除tomcat的依赖，然后再引入Jetty的：

~~~xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
~~~

使用Undertow作为servlet容器时也类似：

~~~xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
~~~

## 嵌入式servlet容器自动配置原理

### 自动配置类分析

嵌入式的Servlet容器自动配置是由EmbeddedServletContainerAutoConfiguration类来管理的：

在这个类中主要完成了几个工作：

1、用@Import导入了BeanPostProcessorsRegistrar类，这个类给容器中导入了一些组件，其中有导入了EmbeddedServletContainerCustomizerBeanPostProcessor类，这个类是EmbeddedServletContainerCustomizer相关的后置处理器。

2、在EmbeddedTomcat内部类中通过@ConditionalOnClass注解判断当前是否引入了tomcat依赖，同时判断判断当前容器没有用户自己定义EmbeddedServletContainerFactory，如果条件通过就给容器中加入TomcatEmbeddedServletContainerFactory类，这个类是tomcat嵌入式容器的工厂类。

3、同理，如果使用其他servlet容器，如Jetty或Undertow也有对应的配置类生效，并把对应的工厂类加入容器中。

~~~java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
//导入BeanPostProcessorsRegistrar：Spring注解版；给容器中导入一些组件
//导入了EmbeddedServletContainerCustomizerBeanPostProcessor：
//后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作
public class EmbeddedServletContainerAutoConfiguration {
    
    @Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })//判断当前是否引入了Tomcat依赖；
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)//判断当前容器没有用户自己定义EmbeddedServletContainerFactory：嵌入式的Servlet容器工厂；作用：创建嵌入式的Servlet容器
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}
    
    /**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
			WebAppContext.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedJetty {

		@Bean
		public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
			return new JettyEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedUndertow {

		@Bean
		public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
			return new UndertowEmbeddedServletContainerFactory();
		}

	}
~~~

嵌入式Servlet容器工厂EmbeddedServletContainerFactory：只有一个方法就是获取对应的容器

~~~java
public interface EmbeddedServletContainerFactory {

   //获取嵌入式的Servlet容器
   EmbeddedServletContainer getEmbeddedServletContainer(
         ServletContextInitializer... initializers);

}
~~~

三种自动配置的容器工厂类是它的实现类：

![搜狗截图20180302144835](D:\笔记\springboot\搜狗截图20180302144835.png)

而它返回的容器也有三种实现方式：

![搜狗截图20180302144910](D:\笔记\springboot\搜狗截图20180302144910.png)

### 工厂类分析

以TomcatEmbeddedServletContainerFactory为例分析：

~~~java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
      ServletContextInitializer... initializers) {
    //创建一个Tomcat
   Tomcat tomcat = new Tomcat();
    
    //配置Tomcat的基本环节
   File baseDir = (this.baseDirectory != null ? this.baseDirectory
         : createTempDir("tomcat"));
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
    
    //将配置好的Tomcat传入进去，返回一个EmbeddedServletContainer；并且启动Tomcat服务器
   return getTomcatEmbeddedServletContainer(tomcat);
}
~~~

### 后置处理器分析

后置处理器用来在bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作，以EmbeddedServletContainerCustomizerBeanPostProcessor为例：

初始化bean的时候会检查是否是ConfigurableEmbeddedServletContainer，如果是就往下执行，获取所有的定制器EmbeddedServletContainerCustomizer，然后执行里面的customize方法，填充用户自定义的配置：

~~~java
//初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    //如果当前初始化的是一个ConfigurableEmbeddedServletContainer类型的组件
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
       //
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
    //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            //从容器中获取所有这个类型的组件：EmbeddedServletContainerCustomizer
            //定制Servlet容器，给容器中可以添加一个EmbeddedServletContainerCustomizer类型的组件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}
~~~

### 总结

嵌入式servlet容器配置原理大致如下：

1）、SpringBoot根据导入的依赖情况，给容器中添加相应的EmbeddedServletContainerFactory【TomcatEmbeddedServletContainerFactory】

2）、容器中某个组件要创建对象就会惊动后置处理器；EmbeddedServletContainerCustomizerBeanPostProcessor；

只要是嵌入式的Servlet容器工厂，后置处理器就工作；

3）、后置处理器，从容器中获取所有的EmbeddedServletContainerCustomizer，调用定制器的定制方法，完成用户自定义的配置，对应我们说过的往容器中放入一个EmbeddedServletContainerCustomizer完成配置（修改配置文件中的配置一样对应ServerProperties类，它就是一个EmbeddedServletContainerCustomizer）

## 嵌入式Servlet容器启动原理

嵌入式Servlet容器启动过程：

1、SpringBoot应用启动运行run方法

2、执行refreshContext(context);  SpringBoot刷新IOC容器【创建IOC容器对象，并初始化容器，创建容器中的每一个组件】；如果是web应用创建AnnotationConfigEmbeddedWebApplicationContext，否则创建AnnotationConfigApplicationContext

3、refresh(context);  刷新刚才创建好的ioc容器：

~~~java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
~~~

这其中调用了web的ioc容器的onRefresh方法，在该方法中创建嵌入式的Servlet容器：

createEmbeddedServletContainer();

4、获取嵌入式的Servlet容器工厂：

EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();

该方法从ioc容器中获取EmbeddedServletContainerFactory 组件；TomcatEmbeddedServletContainerFactory创建对象，通过后置处理器触发识别该对象，然后就获取所有的定制器来先定制Servlet容器的相关配置；

5、使用容器工厂获取嵌入式的Servlet容器：

this.embeddedServletContainer = containerFactory      .getEmbeddedServletContainer(getSelfInitializer());

6、嵌入式的Servlet容器创建对象并启动Servlet容器；

先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来

## 使用外置的Servlet容器

嵌入式Servlet容器：应用打成可执行的jar

​		优点：简单、便携；

​		缺点：默认不支持JSP、优化定制比较复杂（可以使用定制器【ServerProperties、自定义EmbeddedServletContainerCustomizer】，也可以自己编写嵌入式Servlet容器的创建工厂【EmbeddedServletContainerFactory】）；

外置的Servlet容器：外面安装Tomcat---应用war包的方式打包，然后放入容器中运行；

### 配置步骤

1）、必须创建一个war项目；（利用idea创建好目录结构）

2）、将嵌入式的Tomcat指定为provided；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

3）、必须编写一个SpringBootServletInitializer的子类，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       //传入SpringBoot应用的主程序
      return application.sources(SpringBoot04WebJspApplication.class);
   }

}
```

4）、启动web服务器就可以使用（需要在idea中事先配置好web服务器）；

### 原理

对比一下嵌入式和非嵌入式的启动方式：

1、嵌入式：jar包，执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器

2、非嵌入式：启动服务器，服务器启动SpringBoot应用【SpringBootServletInitializer】，启动ioc容器；

在servlet3.0规范中的8.2.4 Shared libraries / runtimes pluggability中规定：

1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

3）、还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；

外置的容器启动过程如下：

1、启动Tomcat容器时根据规范去指定目录下寻找ServletContainerInitializer，在spring的web模块就有这个类，根据指引会找到org.springframework.web.SpringServletContainerInitializer类

2、SpringServletContainerInitializer类中有@HandlesTypes(WebApplicationInitializer.class)注解，容器会自动加载所有WebApplicationInitializer.class类的对象，然后传入其中的onStartup方法的参数Set<Class<?>>中

3、每一个WebApplicationInitializer都调用自己的onStartup，这个时候相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

4、SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

~~~java
protected WebApplicationContext createRootApplicationContext(
      ServletContext servletContext) {
    //1、创建SpringApplicationBuilder
   SpringApplicationBuilder builder = createSpringApplicationBuilder();
   StandardServletEnvironment environment = new StandardServletEnvironment();
   environment.initPropertySources(servletContext, null);
   builder.environment(environment);
   builder.main(getClass());
   ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
   if (parent != null) {
      this.logger.info("Root context already created (using as parent).");
      servletContext.setAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
      builder.initializers(new ParentContextApplicationContextInitializer(parent));
   }
   builder.initializers(
         new ServletContextApplicationContextInitializer(servletContext));
   builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
   builder = configure(builder);
    
    //使用builder创建一个Spring应用
   SpringApplication application = builder.build();
   if (application.getSources().isEmpty() && AnnotationUtils
         .findAnnotation(getClass(), Configuration.class) != null) {
      application.getSources().add(getClass());
   }
   Assert.state(!application.getSources().isEmpty(),
         "No SpringApplication sources have been defined. Either override the "
               + "configure method or add an @Configuration annotation");
   // Ensure error pages are registered
   if (this.registerErrorPageFilter) {
      application.getSources().add(ErrorPageFilterConfiguration.class);
   }
    //启动Spring应用
   return run(application);
}
~~~

5、启动spring应用的时候就加载并创建ioc容器：

~~~java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       
       //刷新IOC容器
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
~~~

# springboot与数据访问

## JDBC

需要导入相关的start和数据库驱动：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
~~~

然后在配置文件中配置数据库相关的设置：

~~~yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
~~~

这些配置最后都会映射到DataSourceProperties里面。

然后自动注入DataSource即可使用。（springboot还配置好了JdbcTemplate，注入后也可以直接使用）

自动配置原理：

自动配置类是DataSourceConfiguration(2.0 DataSourceAutoConfiguration ?)，在这个类中会进行判断到底最后哪个数据源生效，springboot默认是用org.apache.tomcat.jdbc.pool.DataSource作为数据源，默认使用Tomcat连接池，springboot支持多种数据源，可以使用spring.datasource.type指定自定义的数据源类型：

~~~java
/**
 * Generic DataSource configuration.
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
       //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
      return properties.initializeDataSourceBuilder().build();
   }

}
~~~

springboot启动时还可以初始化数据库，相关的执行在DataSourceInitializer类完成，其中的runSchemaScripts()；运行建表语句，runDataScripts();运行插入数据的sql语句；

只需要将文件命名为以下两种即可在启动时自动执行其中的语句：

~~~
schema-*.sql、data-*.sql
~~~

还可以指定要执行的文件：

~~~yaml
spring:
  datasource:
  	schema:
      - classpath:department.sql
~~~

这样就相当于启动时执行类路径下的department.sql了。

## 整合Druid数据源

Druid数据源需要改变配置文件中数据源的类型：

~~~yaml
spring:
  datasource:
  	type: com.alibaba.druid.pool.DruidDataSource
~~~

然后再重新启动就可以完成数据源的切换了，但是如果要配置更多的信息如：

~~~yaml
spring:
  datasource:
  	initialSize: 5
~~~

此时这个配置项不会自动被springboot识别，因为映射不到对应的类（新版本的可能已经可以了），此时需要自定义一个配置类，在其中自己new一个数据源然后放入容器：

~~~java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }
}
~~~

这样配置文件中的配置（spring.datasource开头的）就可以自动读到了。

使用Druid时可以很方便的监控数据交互情况，此时需要配置对应的servlet和filter（也可以通过配置文件的方式实现，但是往容器里放这些对象更简洁）：

~~~java
//配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
~~~

这里的方法名都是固定的，然后再通过浏览器访问/druid路径，输入正确的用户名和密码就能访问到监控后台了。

也可以直接引入Druid的starter来实现：

~~~xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
        </dependency>
~~~

然后在配置文件中定制相关配置。

SpringBoot配置示例

<https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter>

配置项列表<https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8>

## 整合MyBatis

首先要引入starter，官方starter的格式和第三方不一样，第三方的都会将三方件名字写在前面

SpringBoot官方的Starter：spring-boot-starter-*

第三方的： *-spring-boot-starter 

~~~xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
~~~

MybatisAutoConfiguration是对应的自动配置类：

~~~java
@EnableConfigurationProperties(MybatisProperties.class) ： MyBatis配置项绑定类。
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration{}
~~~

MyBatis配置都被绑定在MybatisProperties：

~~~java
@ConfigurationProperties(prefix = "mybatis")
public class MybatisProperties
~~~

相关配置都可以写在springboot配置文件中，相关配置前缀为mybatis：

~~~yaml
# 配置mybatis规则
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射文件位置
~~~

几个关键的类：

- SqlSessionFactory: 自动配置好了
- SqlSession：自动配置了 **SqlSessionTemplate 组合了SqlSession**


- @Import(**AutoConfiguredMapperScannerRegistrar**.**class**）；
- Mapper： 只要我们写的操作MyBatis的接口标准了 **@Mapper 就会被自动扫描进来**，主要使用方式就是自动注入mapper，然后通过映射配置文件，直接调用mapper的方法，操作数据库

（@MapperScan("com.atguigu.admin.mapper") 简化，其他的接口就可以不用标注@Mapper注解）

映射文件：

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.admin.mapper.AccountMapper">
<!--    public Account getAcct(Long id); -->
    <select id="getAcct" resultType="com.atguigu.admin.bean.Account">
        select * from  account_tbl where  id=#{id}
    </select>
</mapper>
~~~

全局配置文件也可以不配置，相关作用可以直接用springboot配置文件代替：

~~~yaml
# 配置mybatis规则
mybatis:
#  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
    
 可以不写全局；配置文件，所有全局配置文件的配置都放在configuration配置项中即可
~~~

mybatis可以用注解版，也可以同时使用配置：

~~~java
//指定这是一个操作数据库的mapper
//@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(department_name) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set department_name=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
~~~

## 整合redis

首先导入redis的starter：

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
~~~

默认会导入lettuce的客户端操作redis：

自动配置：

- RedisAutoConfiguration 自动配置类。RedisProperties 属性类 --> **spring.redis.xxx是对redis的配置**
- 连接工厂是准备好的。**Lettuce**ConnectionConfiguration、**Jedis**ConnectionConfiguration


- **自动注入了RedisTemplate**<**Object**, **Object**> ： xxxTemplate；
- **自动注入了StringRedisTemplate；k：v都是String**


- **key：value**
- **底层只要我们使用 ****StringRedisTemplate、****RedisTemplate就可以操作redis**

使用redis只需要自动注入RedisTemplate，然后调用集合，就能直接操作redis：

~~~java
    @Test
    void testRedis(){
        ValueOperations<String, String> operations = redisTemplate.opsForValue();

        operations.set("hello","world");

        String hello = operations.get("hello");
        System.out.println(hello);
    }
~~~

如果想切换到jedis操作redis，需要在配置文件中指定客户端类型为jedis，并且导入jedis的依赖：

~~~java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

<!--        导入jedis-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
~~~

~~~xml
spring:
  redis:
      host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com
      port: 6379
      password: lfy:Lfy123456
      client-type: jedis
      jedis:
        pool:
          max-active: 10
~~~

# 单元测试

**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库**

作为最新版本的JUnit框架，JUnit5与之前版本的Junit框架有很大的不同。由三个不同子项目的几个不同模块组成。

**JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

**JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

**JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部 包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎。

![afeae670-6be9-11e9-8b0d-d3a853e66b8e](D:\笔记\springboot\afeae670-6be9-11e9-8b0d-d3a853e66b8e.png)

注意：

**SpringBoot 2.4 以上版本移除了默认对 ****Vintage 的依赖。如果需要兼容junit4需要自行引入（不能使用junit4的功能 @Test****）

JUnit 5’s Vintage Engine Removed from **spring-boot-starter-test,如果需要继续兼容junit4需要自行引入vintage：

~~~xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
~~~

要引入springboot的单元测试要引入对应的starter：

~~~xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
~~~

只需要在UT中加上springboot的注解就可以运行springboot的test，可以用自动注解将容器中的对象注入：

~~~java
@SpringBootTest
class Boot05WebAdminApplicationTests {


    @Test
    void contextLoads() {

    }
}

~~~

（在junit4的时候还必须要加上@SpringBootTest + @RunWith(SpringTest.class)）

springboot单元测试的优点：

- Junit类具有Spring的功能，@Autowired、比如 @Transactional 标注测试方法，测试完成后自动回滚

## junit5常用注解

JUnit5的注解与JUnit4的注解有所变化

<https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations>

- **@Test :**表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest :**表示方法是参数化测试，下方会有详细介绍


- **@RepeatedTest :**表示方法可重复执行，下方会有详细介绍（标注在方法上）
- **@DisplayName :**为测试类或者测试方法设置展示名称（运行UT时会显示这个注解内的说明，可以标注在类和方法上）


- **@BeforeEach :**表示在每个单元测试之前执行
- **@AfterEach :**表示在每个单元测试之后执行


- **@BeforeAll :**表示在所有单元测试之前执行
- **@AfterAll :**表示在所有单元测试之后执行


- **@Tag :**表示单元测试类别，类似于JUnit4中的@Categories
- **@Disabled :**表示测试类或测试方法不执行，类似于JUnit4中的@Ignore


- **@Timeout :**表示测试方法运行如果超过了指定时间将会返回错误（标注在方法上）
- **@ExtendWith :**为测试类或测试方法提供扩展类引用（@SpringBootTest就是扩展自这个注解）

~~~java
import org.junit.jupiter.api.Test; //注意这里使用的是jupiter的Test注解！！


public class TestDemo {

  @Test
  @DisplayName("第一次测试")
  public void firstTest() {
      System.out.println("hello world");
  }
~~~

## 断言

数组断言：

~~~java
@Test
@DisplayName("array assertion")
public void array() {
 assertArrayEquals(new int[]{1, 2}, new int[] {1, 2});
}
~~~

组合断言，多个断言代表测试成功：

~~~java
@Test
@DisplayName("assert all")
public void all() {
 assertAll("Math",
    () -> assertEquals(2, 1 + 1),
    () -> assertTrue(1 > 0)
 );
}
~~~

异常断言：

~~~java
@Test
@DisplayName("异常测试")
public void exceptionTest() {
    ArithmeticException exception = Assertions.assertThrows(
           //扔出断言异常
            ArithmeticException.class, () -> System.out.println(1 % 0));

}
~~~

超时断言：

~~~java
@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
~~~

快速失败：

~~~java
@Test
@DisplayName("fail")
public void shouldFail() {
 fail("This should fail");
}
~~~

## 前置条件

JUnit 5 中的前置条件（**assumptions【假设】**）类似于断言，不同之处在于它失败之后不会导致测试方法失败，而是会跳过不执行该测试方法：

~~~java
Assumptions.assumeTrue(true, "终止执行UT");
~~~

~~~java
@DisplayName("前置条件")
public class AssumptionsTest {
 private final String environment = "DEV";
 
 @Test
 @DisplayName("simple")
 public void simpleAssume() {
    assumeTrue(Objects.equals(this.environment, "DEV"));
    assumeFalse(() -> Objects.equals(this.environment, "PROD"));
 }
 
 @Test
 @DisplayName("assume then do")
 public void assumeThenDo() {
    assumingThat(
       Objects.equals(this.environment, "DEV"),
       () -> System.out.println("In DEV")
    );
 }
}
~~~

assumeTrue 和 assumFalse 确保给定的条件为 true 或 false，不满足条件会使得测试执行终止。assumingThat 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止。

## 嵌套测试

JUnit 5 可以通过 Java 中的内部类和@Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。

嵌套类的@BeforeEach和@AfterEach可以影响本类及其内部类的测试方法，可以通过这个规则来控制测试方法的执行顺序

~~~java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
~~~

## 参数化测试

参数化测试可以很方便的指定多个UT的入参，这样就不用再不同入参写多个测试方法了，节省代码量

**@ValueSource**: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型

**@NullSource**: 表示为参数化测试提供一个null的入参

**@EnumSource**: 表示为参数化测试提供一个枚举入参

**@CsvFileSource**：表示读取指定CSV文件内容作为参数化测试入参

**@MethodSource**：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)

下面是ValueSource和MethodSource的例子：

~~~java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}


@ParameterizedTest
@MethodSource("method")    //指定方法名
@DisplayName("方法来源参数")
public void testWithExplicitLocalMethodSource(String name) {
    System.out.println(name);
    Assertions.assertNotNull(name);
}

static Stream<String> method() {
    return Stream.of("apple", "banana");
}
~~~

## 将junit4迁移到5的变化

在进行迁移的时候需要注意如下的变化：

- 注解在 org.junit.jupiter.api 包中，断言在 org.junit.jupiter.api.Assertions 类中，前置条件在 org.junit.jupiter.api.Assumptions 类中。
- 把@Before 和@After 替换成@BeforeEach 和@AfterEach。


- 把@BeforeClass 和@AfterClass 替换成@BeforeAll 和@AfterAll。
- 把@Ignore 替换成@Disabled。


- 把@Category 替换成@Tag。
- 把@RunWith、@Rule 和@ClassRule 替换成@ExtendWith。

# 指标监控

未来每一个微服务在云上部署以后，我们都需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

1.x与2.x的不同：

![image (13)](D:\笔记\springboot\image (13).png)

使用方法很简单，就是引入starter，然后配置对应的配置文件：

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
~~~

~~~yaml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露
~~~

项目启动后，直接访问对应的url就能得到相应的监控信息：

<http://localhost:8080/actuator/beans>

<http://localhost:8080/actuator/configprops>

<http://localhost:8080/actuator/metrics>

<http://localhost:8080/actuator/metrics/jvm.gc.pause>

[http://localhost:8080/actuator/](http://localhost:8080/actuator/metrics)endpointName/detailPath

## Actuator Endpoint

1、常用的检查端点

| ID                 | 描述                                       |
| ------------------ | ---------------------------------------- |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。               |
| `caches`           | 暴露可用的缓存。                                 |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。               |
| `configprops`      | 显示所有`@ConfigurationProperties`。          |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`     |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。需要一个或多个`Flyway`组件。   |
| `health`           | 显示应用程序运行状况信息。                            |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                         |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                         |
| `mappings`         | 显示所有`@RequestMapping`路径列表。               |
| `scheduledtasks`   | 显示应用程序中的计划任务。                            |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                          |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                  |

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加端点：

| ID           | 描述                                       |
| ------------ | ---------------------------------------- |
| `heapdump`   | 返回`hprof`堆转储文件。                          |
| `jolokia`    | 通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP`Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要依赖`micrometer-registry-prometheus`。 |

其中最常用的Endpoint

- **Health：监控状况**
- **Metrics：运行时指标**


- **Loggers：日志记录**

2、Health Endpoint

健康检查端点，我们一般用于在云平台，平台会定时的检查应用的健康状况，我们就需要Health Endpoint可以为平台返回当前应用的一系列组件健康状况的集合。

重要的几点：

- health endpoint返回的结果，应该是一系列健康检查后的一个汇总报告
- 很多的健康检查默认已经自动配置好了，比如：数据库、redis等


- 可以很容易的添加自定义的健康检查机制

只需要访问http://localhost:8080/actuator/health

![image (14)](D:\笔记\springboot\image (14).png)

3、Metrics Endpoint

提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到；

- 通过Metrics对接多种监控系统
- 简化核心Metrics开发


- 添加自定义Metrics或者扩展已有Metrics

![image (15)](D:\笔记\springboot\image (15).png)

4、管理endpoint

- 默认所有的Endpoint除过shutdown都是开启的。
- 需要开启或者禁用某个Endpoint，需要修改配置文件，如开启beans的endpoint：

~~~yaml
management:
  endpoint:
    beans:
      enabled: true
~~~

- 或者禁用所有的Endpoint然后手动开启指定的Endpoint：

~~~yaml
management:
  endpoints:
    enabled-by-default: false
  endpoint:
    beans:
      enabled: true
    health:
      enabled: true
~~~

暴露endpoint的方式主要有HTTP和JMS两种：

- HTTP：默认只暴露**health**和**info **Endpoint
- **JMX**：默认暴露所有Endpoint（也就是所有监控指标都可以通过jconsole可见）


- 除过health和info，剩下的Endpoint都应该进行保护访问。如果引入SpringSecurity，则会默认配置安全访问规则

## 定制 Endpoint

1、定制 Health 信息

可以设置健康状态与detail：

~~~java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }


        builder.withDetail("code",100)
                .withDetails(map);

    }
}
~~~

2、定制info信息

常用两种方式：

（1）编写配置文件

~~~yaml
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
~~~

（2）编写InfoContributor

~~~java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
~~~

最后访问的结果是两种方式监控值的并集

3、定制Metrics信息

可以通过扩展监控下列指标：

- JVM metrics, report utilization of:


- - Various memory and buffer pools
  - Statistics related to garbage collection


- - Threads utilization
  - Number of classes loaded/unloaded


- CPU metrics
- File descriptor metrics


- Kafka consumer and producer metrics
- Log4j2 metrics: record the number of events logged to Log4j2 at each level


- Logback metrics: record the number of events logged to Logback at each level
- Uptime metrics: report a gauge for uptime and a fixed gauge representing the application’s absolute start time


- Tomcat metrics (`server.tomcat.mbeanregistry.enabled` must be set to `true` for all Tomcat metrics to be registered)
- [Spring Integration](https://docs.spring.io/spring-integration/docs/5.4.1/reference/html/system-management.html#micrometer-integration) metrics

下面是定制metrics的例子，分别是监控某个变量的值和集合的size：

~~~java
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }

    public void hello() {
        counter.increment();
    }
}


//也可以使用下面的方式
@Bean
MeterBinder queueSize(Queue queue) {
    return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
}
~~~

4、定制一个endpoint

需要指定endpoint的名称，实现一个读方法，一个写方法：

~~~java
@Component
@Endpoint(id = "container")
public class DockerEndpoint {


    @ReadOperation
    public Map getDockerInfo(){
        return Collections.singletonMap("info","docker started...");
    }

    @WriteOperation
    private void restartDocker(){
        System.out.println("docker restarted....");
    }

}
~~~

# 缓存

JSR107是java缓存的规范，Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry和Expiry：

- CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
- CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
- Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
- Entry是一个存储在Cache中的key-value对。
- Expiry每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

![QQ图片20211002102118](D:\笔记\springboot\QQ图片20211002102118.png)

Spring从3.1开始定义了org.springframework.cache.Cache，和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发；

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
- Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；

每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

使用Spring缓存抽象时我们需要关注以下两点：

- 1、确定方法需要被缓存以及他们的缓存策略
- 2、从缓存中读取之前缓存存储的数据

## @Cacheable

要使用spring缓存需要先引入对应starter：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
~~~

然后在启动类上加上@EnableCaching

~~~java
@MapperScan("com.atguigu.cache.mapper")
@SpringBootApplication
@EnableCaching
public class Springboot01CacheApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot01CacheApplication.class, args);
	}
}
~~~

在方法上标注@Cacheable代表将方法的运行结果缓存起来，下次访问相同的数据直接从缓存中取：

~~~java
@Cacheable(value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/)
    public Employee getEmp(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }
~~~

注解的几个属性：

cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；

key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值

​          可以编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0] getEmp[2]

keyGenerator：key的生成器；可以自己指定key的生成器的组件id

key/keyGenerator：二选一使用;

cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
condition：指定符合条件的情况下才缓存；
	如：condition = "#id>0"
	condition = "#a0>1"：第一个参数的值》1的时候才进行缓存

unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
		如 unless = "#result == null"
		unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
sync：是否使用异步模式

一、原理

1、自动配置类；CacheAutoConfiguration
 2、缓存的配置类：会将这些缓存配置类放入容器中
 org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
 org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
 org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
 org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
 org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
 org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
 org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
 org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
 org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
 org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
 org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
 3、哪个配置类默认生效：SimpleCacheConfiguration；

 4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
 5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；

 运行流程：
 @Cacheable：
 1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
	（CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
 2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
	key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
		SimpleKeyGenerator生成key的默认策略；
				如果没有参数；key=new SimpleKey()；
				如果有一个参数：key=参数的值
				如果有多个参数：key=new SimpleKey(params)；
 3、没有查到缓存就调用目标方法；
 4、将目标方法返回的结果，放进缓存中

 @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
 如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；

 核心：
	1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
	2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
二、自定义的KeyGenerator，在使用@Cacheable时可以指定自己的key生成逻辑

~~~java
package com.atguigu.cache.config;

import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.Method;
import java.util.Arrays;

@Configuration
public class MyCacheConfig {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+ Arrays.asList(params).toString()+"]";
            }
        };
    }
}

~~~

## 其他缓存注解

1、@CachePut

它是用来刷新缓存的，既调用方法，又更新缓存数据；

运行的步骤就是先调用方法，后把目标方法返回的结果缓存起来

~~~java
@CachePut(/*value = "emp",*/key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
~~~

注意@Cacheable的key是不能用#result的，因为它要求在方法调用前查缓存，此时是没有返回结果result的

2、@CacheEvict

它用来清除缓存

key：指定要清除的数据
allEntries = true：指定清除这个缓存中所有的数据
beforeInvocation = false：缓存的清除是否在方法之前执行

默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除


beforeInvocation = true：

代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除

~~~java
@CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        int i = 10/0;
    }
~~~

3、@Caching

它是上面三个注解的集合体，用来定义复杂的缓存规则：

~~~java
@Caching(
         cacheable = {
             @Cacheable(/*value="emp",*/key = "#lastName")
         },
         put = {
             @CachePut(/*value="emp",*/key = "#result.id"),
             @CachePut(/*value="emp",*/key = "#result.email")
         }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
~~~

## 用redis做缓存

导入redis的依赖之后，起作用的就是RedisCacheConfiguration了，此时底层就不再使用默认的map做缓存，而是用redis了。

用redis做缓存时要注意指定序列化，否则当缓存对象时，redis存的就是二进制数据，所以我们要给redis设置序列化相关，并把序列化的设置绑定在cacheManager中：（一个对象一个RedisCacheManager）

~~~java
@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }



    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary  //将某个缓存管理器作为默认的
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }

    @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }


}
~~~

然后在类或者方法上指定使用什么cacheManager：

~~~java
@CacheConfig(cacheNames="emp",cacheManager = "employeeCacheManager")
~~~

也可以直接将cacheManager注入到类中直接使用：

~~~java
@Qualifier("deptCacheManager")
    @Autowired
    RedisCacheManager deptCacheManager;

public Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);

        //获取某个缓存
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("dept:1",department);

        return department;
    }
~~~

# 消息队列

消息服务中间件来提升系统异步通信、扩展解耦能力

消息服务中两个重要概念：
消息代理（message broker）和目的地（destination）
当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

消息队列主要有两种形式的目的地：
1.队列（queue）：点对点消息通信（point-to-point）（消息只有唯一的发送者和接受者）
2.主题（topic）：发布（publish）/订阅（subscribe）消息通信；多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

消息服务的两个重要协议：

1、JMS（Java Message Service）JAVA消息服务：
–基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现
2、AMQP（Advanced Message Queuing Protocol）
–高级消息队列协议，也是一个消息代理的规范，兼容JMS
–RabbitMQ是AMQP的实现

两个协议的对比：

![QQ图片20211002212127](D:\笔记\springboot\QQ图片20211002212127.png)

spring本身支持的消息服务：

spring-jms提供了对JMS的支持
–spring-rabbit提供了对AMQP的支持
–需要ConnectionFactory的实现来连接消息代理
–提供JmsTemplate、RabbitTemplate来发送消息
–@JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
–@EnableJms、@EnableRabbit开启支持

## RabbitMQ简介

AMQP 中的消息路由
•AMQP 中消息的路由过程和Java 开发者熟悉的JMS 存在一些差别，AMQP 中增加了Exchange和Binding的角色。生产者把消息发布到Exchange 上，消息最终到达队列并被消费者接收，而Binding 决定交换器的消息应该发送到那个队列：

![QQ图片20211002212415](D:\笔记\springboot\QQ图片20211002212415.png)

四种exchange类型：（headers 匹配AMQP 消息的header 而不是路由键，headers 交换器和direct 交换器完全一致，但性能差很多，目前几乎用不到了）

1、完全匹配的direct类型：

消息中的路由键（routing key）如果和Binding 中的binding key 一致，交换器就将消息发到对应的队列中

![QQ图片20211002212626](D:\笔记\springboot\QQ图片20211002212626.png)

2、fanout广播类型：

每个发到fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

![QQ图片20211002212736](D:\笔记\springboot\QQ图片20211002212736.png)

3、支持通配符的匹配topic

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，*匹配一个单词。

![QQ图片20211002212938](D:\笔记\springboot\QQ图片20211002212938.png)

## RabbitMQ整合

首先引入对应的starter：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
~~~

在配置文件中配置消息中间件的信息：

~~~properties
spring.rabbitmq.host=118.24.44.169
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
~~~

然后在消息中间件那一边创建好一些exchange、队列，并且将它们绑定。

要使用只需要自动注入rabbitTemplate，通过它就可以完成消息的发送和接受：

~~~java
@Autowired
	RabbitTemplate rabbitTemplate;

发送数据：
//Message需要自己构造一个;定义消息体内容和消息头
//rabbitTemplate.send(exchage,routeKey,message);

//object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq；
//rabbitTemplate.convertAndSend(exchage,routeKey,object);

接受数据：
Object o = rabbitTemplate.receiveAndConvert("atguigu.news");

广播：只需要改变exchange，此时不需要routekey，广播不关心
rabbitTemplate.convertAndSend("exchange.fanout","",new Book("红楼梦","曹雪芹"));
~~~

这个时候如果发送和接受的信息是一个对象，底层默认就会调用java的序列化，如果我们想自己定制序列化规则就可以替换它：

~~~java
@Configuration
public class MyAMQPConfig {

    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
~~~

然后再发送和接受对象时，底层就会自动帮我们序列化：

~~~java
//对象被默认序列化以后发送出去，如果有自定义的序列化就调用自己的
rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",new Book("西游记","吴承恩"));
~~~

此外我们还可以用注解的方式监听消息，首先需要在启动类上开启注解设置@EnableRabbit：

~~~java
@EnableRabbit  //开启基于注解的RabbitMQ模式
@SpringBootApplication
public class Springboot02AmqpApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot02AmqpApplication.class, args);
	}
}
~~~

然后就可以通过注解来接受消息了，有对应消息时就会自动调用方法并传入入参消息

~~~java
@Service
public class BookService {

    @RabbitListener(queues = "atguigu.news")
    public void receive(Book book){
        System.out.println("收到消息："+book);
    }

    @RabbitListener(queues = "atguigu")
    public void receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}
~~~

除此之外我们还可以注入AmqpAdmin来管理mq，完成exchange、queue、及其绑定设置：

~~~java
@Autowired
	AmqpAdmin amqpAdmin;

	@Test
	public void createExchange(){

//		amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
//		System.out.println("创建完成");

//		amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
		//创建绑定规则

//		amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));

		//amqpAdmin.de
	}
~~~

# 检索

##ElasticSearch简介

我们的应用经常需要添加检索功能，开源的ElasticSearch是目前全文搜索引擎的首选。他可以快速的存储、搜索和分析海量数据。

Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能，github等大型的站点也是采用了ElasticSearch作为其搜索服务

一个ElasticSearch 集群可以包含多个索引，相应的每个索引可以包含多个类型。这些不同的类型存储着多个文档，每个文档又有多个属性。
•类似关系：
–索引-数据库
–类型-表
–文档-表中的记录
–属性-列

![QQ图片20211003132615](D:\笔记\springboot\QQ图片20211003132615.png)

可以直接通过postman访问单独部署的ES，完成数据的增删改查。

## ElasticSearch整合

SpringBoot默认支持两种技术来和ES交互；

1、Jest（默认不生效）
需要导入jest的工具包（io.searchbox.client.JestClient）
2、SpringData ElasticSearch【ES版本有可能不合适】

版本适配说明：https://github.com/spring-projects/spring-data-elasticsearch
如果版本不适配：2.4.6

1）、升级SpringBoot版本
2）、安装对应版本的ES

一、Jest方式整合

首先引入依赖：

~~~xml
<dependency>
			<groupId>io.searchbox</groupId>
			<artifactId>jest</artifactId>
			<version>5.3.3</version>
		</dependency>
~~~

然后在配置文件中配置ES的位置：

~~~properties
spring.elasticsearch.jest.uris=http://118.24.44.169:9200
~~~

自动注入JestClient就可以使用了：

~~~java
@Test
	public void contextLoads() {
		//1、给Es中索引（保存）一个文档；
		Article article = new Article();
		article.setId(1);
		article.setTitle("好消息");
		article.setAuthor("zhangsan");
		article.setContent("Hello World");

		//构建一个索引功能
		Index index = new Index.Builder(article).index("atguigu").type("news").build();

		try {
			//执行
			jestClient.execute(index);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	//测试搜索
	@Test
	public void search(){

		//查询表达式
		String json ="{\n" +
				"    \"query\" : {\n" +
				"        \"match\" : {\n" +
				"            \"content\" : \"hello\"\n" +
				"        }\n" +
				"    }\n" +
				"}";

		//更多操作：https://github.com/searchbox-io/Jest/tree/master/jest
		//构建搜索功能
		Search search = new Search.Builder(json).addIndex("atguigu").addType("news").build();

		//执行
		try {
			SearchResult result = jestClient.execute(search);
			System.out.println(result.getJsonString());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
~~~

如果构建的时候不显式指定id，就必须在实体类中用注解指定id：

~~~java
public class Article {

    @JestId
    private Integer id;
    private String author;
    private String title;
    private String content;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
~~~

二、SpringData ElasticSearch整合

首先导入依赖：

~~~xml
<!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		</dependency>
~~~

然后配置：

~~~properties
spring.data.elasticsearch.cluster-name=elasticsearch
spring.data.elasticsearch.cluster-nodes=118.24.44.169:9301
~~~

这种方式需要自定义一个持久化类：泛型类型是存储的类和其主键类型，这个类无需提供实现，只需要根据ES的规则命名方法即可

~~~java
public interface BookRepository extends ElasticsearchRepository<Book,Integer> {

    //参照
    // https://docs.spring.io/spring-data/elasticsearch/docs/3.0.6.RELEASE/reference/html/
   public List<Book> findByBookNameLike(String bookName);

}
~~~

要存储的实体类要标注对应的索引和类型：

~~~java
@Document(indexName = "atguigu",type = "book")
public class Book {
    private Integer id;
    private String bookName;
    private String author;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", bookName='" + bookName + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}
~~~

然后就可以直接调用方法查询：

~~~java
@Autowired
	BookRepository bookRepository;

	@Test
	public void test02(){
//		Book book = new Book();
//		book.setId(1);
//		book.setBookName("西游记");
//		book.setAuthor("吴承恩");
//		bookRepository.index(book);


		for (Book book : bookRepository.findByBookNameLike("游")) {
			System.out.println(book);
		}
		;

	}
~~~

# springboot与任务

## 异步任务

要使用spring的异步任务要先在启动类加上@EnableAsync：

~~~java
@EnableAsync  //开启异步注解功能
@SpringBootApplication
public class Springboot04TaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot04TaskApplication.class, args);
	}
}
~~~

然后在需要异步调用的方法上标注注解即可：

~~~java
@Service
public class AsyncService {

    //告诉Spring这是一个异步方法
    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中...");
    }
}
~~~

## 定时任务

要使用spring的定时任务需要在启动类上加上@EnableScheduling：

~~~java
@EnableScheduling //开启基于注解的定时任务
@SpringBootApplication
public class Springboot04TaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot04TaskApplication.class, args);
	}
}
~~~

然后通过在方法上标注@Scheduled，就可以完成定时任务了：

~~~java
@Service
public class ScheduledService {

    /**
     * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
     * 0 * * * * MON-FRI
     *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
     */
   // @Scheduled(cron = "0 * * * * MON-SAT")
    //@Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT")
   // @Scheduled(cron = "0-4 * * * * MON-SAT")
    @Scheduled(cron = "0/4 * * * * MON-SAT")  //每4秒执行一次
    public void hello(){
        System.out.println("hello ... ");
    }
}

~~~

![QQ图片20211003154155](D:\笔记\springboot\QQ图片20211003154155.png)

## 邮件任务

先引入邮件的依赖spring-boot-starter-mail：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-mail</artifactId>
		</dependency>
~~~

然后在配置中指定自己邮箱的信息：

~~~properties
spring.mail.username=534096094@qq.com
spring.mail.password=gtstkoszjelabijb   //qq邮箱的授权码
spring.mail.host=smtp.qq.com            //qq邮箱的smtp地址
spring.mail.properties.mail.smtp.ssl.enable=true   // 开启ssl
~~~

然后自动注入JavaMailSenderImpl即可完成邮件发送服务：

~~~java
@Autowired
	JavaMailSenderImpl mailSender;

	@Test
	public void contextLoads() {
		SimpleMailMessage message = new SimpleMailMessage();
		//邮件设置
		message.setSubject("通知-今晚开会");
		message.setText("今晚7:30开会");

		message.setTo("17512080612@163.com");
		message.setFrom("534096094@qq.com");

		mailSender.send(message);
	}

	@Test
	public void test02() throws  Exception{
		//1、创建一个复杂的消息邮件
		MimeMessage mimeMessage = mailSender.createMimeMessage();
		MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

		//邮件设置
		helper.setSubject("通知-今晚开会");
		helper.setText("<b style='color:red'>今天 7:30 开会</b>",true);

		helper.setTo("17512080612@163.com");
		helper.setFrom("534096094@qq.com");

		//上传文件
		helper.addAttachment("1.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\1.jpg"));
		helper.addAttachment("2.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\2.jpg"));

		mailSender.send(mimeMessage);

	}
~~~

# springboot与安全

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。几个类：
WebSecurityConfigurerAdapter：自定义Security策略
AuthenticationManagerBuilder：自定义认证策略
@EnableWebSecurity：开启WebSecurity模式

Spring Security 的两个目标：认证和授权

1、“认证”（Authentication），是建立一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统）。

2、“授权”（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

## 认证与授权

首先要引入安全的依赖：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
~~~

然后需要自定义一个配置类，加上安全的注解，代表这个是安全的配置类：

~~~java
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //super.configure(http);
        //定制请求的授权规则
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("VIP1")
                .antMatchers("/level2/**").hasRole("VIP2")
                .antMatchers("/level3/**").hasRole("VIP3");

        //开启自动配置的登陆功能，效果，如果没有登陆，没有权限就会来到登陆页面
        http.formLogin().usernameParameter("user").passwordParameter("pwd")
                .loginPage("/userlogin");
        //1、/login来到登陆页
        //2、重定向到/login?error表示登陆失败
        //3、更多详细规定
        //4、默认post形式的 /login代表处理登陆
        //5、一但定制loginPage；那么 loginPage的post请求就是登陆


        //开启自动配置的注销功能。
        http.logout().logoutSuccessUrl("/");//注销成功以后来到首页
        //1、访问 /logout 表示用户注销，清空session
        //2、注销成功会返回 /login?logout 页面；

        //开启记住我功能
        http.rememberMe().rememberMeParameter("remeber");
        //登陆成功以后，将cookie发给浏览器保存，以后访问页面带上这个cookie，只要通过检查就可以免登录
        //点击注销会删除cookie

    }

    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //super.configure(auth);
        auth.inMemoryAuthentication()
                .withUser("zhangsan").password("123456").roles("VIP1","VIP2")
                .and()
                .withUser("lisi").password("123456").roles("VIP2","VIP3")
                .and()
                .withUser("wangwu").password("123456").roles("VIP1","VIP3");

    }
}
~~~

其中两个方法，从上到下分别是授权和认证。

在授权的方法里可以规定不同角色可以访问的目录，在认证规则中可以规定不同登录用户对应的角色是什么。

此外，登录、注销、记住我的功能也是安全框架提供的功能，它可以将某个页面设置为登录页，如果登录失败跳转到的位置（其实还是认证的一部分实现）；注销是退出登录；记住我是指浏览器关闭后，用户再次访问也无需登录，这个是安全框架基于cookie的一套实现机制。

## Thymeleaf整合安全

首先引入相关的依赖：

~~~xml
<dependency>
			<groupId>org.thymeleaf.extras</groupId>
			<artifactId>thymeleaf-extras-springsecurity4</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
~~~

然后在html上使用对应标签即可完成权限控制：

~~~html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
	  xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1 align="center">欢迎光临武林秘籍管理系统</h1>
<div sec:authorize="!isAuthenticated()">
	<h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
</div>
<div sec:authorize="isAuthenticated()">
	<h2><span sec:authentication="name"></span>，您好,您的角色有：
		<span sec:authentication="principal.authorities"></span></h2>
	<form th:action="@{/logout}" method="post">
		<input type="submit" value="注销"/>
	</form>
</div>

<hr>

<div sec:authorize="hasRole('VIP1')">
	<h3>普通武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level1/1}">罗汉拳</a></li>
		<li><a th:href="@{/level1/2}">武当长拳</a></li>
		<li><a th:href="@{/level1/3}">全真剑法</a></li>
	</ul>

</div>

<div sec:authorize="hasRole('VIP2')">
	<h3>高级武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level2/1}">太极拳</a></li>
		<li><a th:href="@{/level2/2}">七伤拳</a></li>
		<li><a th:href="@{/level2/3}">梯云纵</a></li>
	</ul>

</div>

<div sec:authorize="hasRole('VIP3')">
	<h3>绝世武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level3/1}">葵花宝典</a></li>
		<li><a th:href="@{/level3/2}">龟派气功</a></li>
		<li><a th:href="@{/level3/3}">独孤九剑</a></li>
	</ul>
</div>


</body>
</html>
~~~

其中关键的标签是sec:authorize="!isAuthenticated()"，代表检查是否授权；sec:authorize="hasRole('VIP1')"代表检查是否有这个角色的权限。这样我们就能根据权限的不同，页面直接显示不同的内容

此外，还可以设置特殊的name，和后台功能绑定：

~~~html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1 align="center">欢迎登陆武林秘籍管理系统</h1>
	<hr>
	<div align="center">
		<form th:action="@{/userlogin}" method="post">
			用户名:<input name="user"/><br>
			密码:<input name="pwd"><br/>
			<input type="checkbox" name="remeber"> 记住我<br/>
			<input type="submit" value="登陆">
		</form>
	</div>
</body>
</html>
~~~

如这里的记住我标签的name是remeber，在后台指定该name和安全的记住我关联：

~~~java
http.rememberMe().rememberMeParameter("remeber");
~~~

# 分布式

## dubbo整合springboot

Dubbo是Alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。

![QQ图片20211004103451](D:\笔记\springboot\QQ图片20211004103451.png)

首先要安装zk和dubbo。我们需要准备两个服务，一个是提供者，一个是消费者。

一、首先准备提供者，需要先引入依赖：

~~~xml
<dependency>
			<groupId>com.alibaba.boot</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>0.1.0</version>
		</dependency>
<!--引入zookeeper的客户端工具-->
		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
~~~

在配置文件中配置服务的名字、注册中心的地址、对外开放服务的路径：

~~~properties
dubbo.application.name=provider-ticket
dubbo.registry.address=zookeeper://118.24.44.169:2181
dubbo.scan.base-packages=com.atguigu.ticket.service
~~~

在service下准备对外开放的服务：

~~~java
public interface TicketService {

    public String getTicket();
}
~~~

~~~java
import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;


@Component
@Service //将服务发布出去
public class TicketServiceImpl implements TicketService {
    @Override
    public String getTicket() {
        return "《厉害了，我的国》";
    }
}
~~~

注意，这里的@Service是dubbo提供的注解。要先将提供者运行起来，将服务注册到注册中心

二、提供消费者

引入相同的依赖，在配置文件中指定服务名、配置注册中心的地址：

~~~properties
dubbo.application.name=consumer-user
dubbo.registry.address=zookeeper://118.24.44.169:2181
~~~

然后定义一个接口，和服务提供者对外开放的接口是相同的路径：

~~~java
public interface TicketService {

    public String getTicket();
}
~~~

不必提供实现，而是在用到它的地方加上注解代表它是引用其他服务的即可：

~~~java
import com.alibaba.dubbo.config.annotation.Reference;
import com.atguigu.ticket.service.TicketService;
import org.springframework.stereotype.Service;

@Service
public class UserService{

    @Reference
    TicketService ticketService;

    public void hello(){
        String ticket = ticketService.getTicket();
        System.out.println("买到票了："+ticket);
    }
}
~~~

## springcloud整合springboot

Spring Cloud是一个分布式的整体解决方案。Spring Cloud 为开发者提供了在分布式系统（配置管理，服务发现，熔断，路由，微代理，控制总线，一次性token，全局琐，leader选举，分布式session，集群状态）中快速构建的工具，使用Spring Cloud的开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接。

•SpringCloud分布式开发五大常用组件
•服务发现——Netflix Eureka
•客服端负载均衡——Netflix Ribbon
•断路器——Netflix Hystrix
•服务网关——Netflix Zuul
•分布式配置——Spring Cloud Config

对springcloud的使用分为三步：

一、构建Eureka注册中心服务

首先引入依赖：

~~~xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
~~~

配置文件：

~~~yaml
server:
  port: 8761
eureka:
  instance:
    hostname: eureka-server  # eureka实例的主机名
  client:
    register-with-eureka: false #不把自己注册到eureka上
    fetch-registry: false #不从eureka上来获取服务的注册信息
    service-url:
      defaultZone: http://localhost:8761/eureka/
~~~

在启动类上标注代表是注册中心：

~~~java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
~~~

二、构建服务提供者

首先引入依赖：

~~~xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
~~~

springcloud默认是用http通信的，所以服务提供者项目和普通的web项目一样，也是对外开放接口：

~~~java
@RestController
public class TicketController {


    @Autowired
    TicketService ticketService;

    @GetMapping("/ticket")
    public String getTicket(){
        return ticketService.getTicket();
    }
}
~~~

~~~java
@Service
public class TicketService {

    public String getTicket(){
        System.out.println("8002");
        return "《厉害了，我的国》";
    }
}
~~~

不同之处在于配置文件配置了服务的相关信息：

~~~yaml
server:
  port: 8002
spring:
  application:
    name: provider-ticket


eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
~~~

三：构建服务消费者

引入和上面相同的依赖。

需要在配置文件中指定一些信息：

~~~yaml
spring:
  application:
    name: consumer-user
server:
  port: 8200

eureka:
  instance:
    prefer-ip-address: true # 注册服务的时候使用服务的ip地址
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
~~~

然后在启动类上开启服务发现，并且自定义一个RestTemplate放入容器中

~~~java
@EnableDiscoveryClient //开启发现服务功能
@SpringBootApplication
public class ConsumerUserApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerUserApplication.class, args);
	}

	@LoadBalanced //使用负载均衡机制
	@Bean
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}
}
~~~

然后远程调用就可以用这个RestTemplate来实现了：

~~~java
@RestController
public class UserController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/buy")
    public String buyTicket(String name){
        String s = restTemplate.getForObject("http://PROVIDER-TICKET/ticket", String.class);
        return name+"购买了"+s;
    }
}
~~~

调用时需要指定http协议、地址用服务名就可以代替，/ticket是提供者暴露出来的接口，这样就可以完成远程调用。

# 热部署

在开发中我们修改一个Java文件后想看到效果不得不重启应用，这导致大量时间花费，我们希望不重启应用的情况下，程序可以自动部署（热部署）

只需要引入依赖：

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
		</dependency>
~~~

然后修改完代码之后，按ctrl+f9就可以快速部署

