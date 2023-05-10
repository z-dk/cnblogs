### SpringBoot的核心注解组成？
核心注解：@SpringBootApplication它由3个子注解组成：
**@SpringBootConfiguration**：组合了@Configuration注解，实现配置文件的功能
**@EnableAutoConfiguration**：开启自动配置，也可以通过exclude关闭
**@ComponentScan**：配置扫描的包

### Spring启动做了什么

Spring的启动过程就是其IoC容器的启动过程，对于web程序，IoC容器启动过程即是建立上下文的过程，在web应用中，web容器会提供一个全局的ServletContext上下文环境，ServletContext上下文为Spring IoC提供了一个宿主环境。

Spring的启动过程大致可以分为以下几个步骤：

1. 创建一个BeanFactory，也就是IoC容器，用来存储和管理Bean。
2. 创建一个BeanDefinitionReader，用来读取配置文件或注解，将需要创建Bean的类转换为BeanDefinition。
3. 创建一个ClassPathBeanDefinitionScanner，用来扫描指定的包路径，将符合条件的类转换为BeanDefinition。
4. 注册一些内置的Bean和后置处理器，例如Environment、ConfigurationClassPostProcessor等。
5. 刷新容器，执行一系列的初始化操作，例如加载配置类、扫描组件、注册Bean、解析依赖、执行生命周期方法等。
6. 启动容器，发布事件通知监听器，开始接收请求。

### Spring注册bean的方式

Spring有以下几种注册Bean的方式：

- 使用XML配置文件，通过\<bean>标签定义Bean的属性和依赖。
- 使用注解，通过@Component, @Repository, @Controller, @Service等注解标注类，让Spring自动扫描和注册Bean。
- 使用@Bean注解，通过在配置类中定义方法，返回一个Bean对象，并用@Bean注解标注，让Spring管理该对象。
- 使用@Import注解，通过在配置类中引入其他类或配置类，让Spring注册这些类为Bean。
- 使用FactoryBean接口，通过实现该接口的getObject()方法，返回一个Bean对象，并用@Component或@Bean注解标注该类，让Spring管理该对象。
- 使用BeanDefinitionRegistry接口，通过实现该接口的registerBeanDefinition()方法，手动向容器中注册一个BeanDefinition对象。

**Spring注解和XML配置存在的兼容性和优先级:**

Spring注解和XML配置的兼容性问题主要有以下几点：

- 如果在XML配置文件中使用了context:component-scan标签，那么需要注意扫描的包路径是否和注解配置的包路径有重叠，否则可能会导致Bean的重复定义或覆盖。
- 如果在XML配置文件中使用了**\<import>**标签，那么需要注意导入的其他XML配置文件是否和注解配置的类有冲突，否则可能会导致Bean的重复定义或覆盖。
- 如果在XML配置文件中使用了**aop:config**标签，那么需要注意切面的定义是否和注解配置的切面有冲突，否则可能会导致切面的重复执行或失效。

Spring注解和XML配置的优先级问题主要有以下几点：

- 如果一个Bean同时在注解配置和XML配置中定义了，那么默认情况下，**XML配置会覆盖注解配置**。
- 如果一个Bean在注解配置中使用了**@Primary或@Order**注解，那么它会优先于其他没有这些注解的Bean被选择。
- 如果一个Bean在XML配置中使用了**primary="true"或depends-on="…"**属性，那么它会优先于其他没有这些属性的Bean被选择。

### SpringBoot自动配置的原理?
自动配置的注解是@EnableAutoConfiguration，从上面的@Import注解可以找到加载自动配置的映射；
SpringFactoriesLoader.loadFactoryNames方法会加载类路径及所有jar包下META-INF/spring.factories配置中映射的自动配置的类；
@Configuration,@ConditionalOnClass就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置；

### SpringBoot Starter启动器是什么？
它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring及其他技术，而不需要到处找示例代码和依赖包；
SpringBoot官方的启动器命名都是spring-boot-starter-xxx，第三方一般是xxx-spring-boot-starter；

### 如何在SpringBoot启动的时候运行一些特定的代码？
可以实现接口 ApplicationRunner或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个run方法；

### SpringBoot有几种读取配置的方式？
SpringBoot可以通过 @PropertySource,@Value,@Environment, @ConfigurationProperties 来绑定变量；

### SpringBoot支持的日志框架有什么？
SpringBoot支持Java Util Logging,Log4j2,Lockback作为日志框架，如果你使用starters启动器，Spring Boot将使用Logback作为默认日志框架；

### spring-boot-maven-plugin有什么用？
它提供了一些像jar一样打包或者运行应用程序的命令

### spring-boot-starter-parent有什么用？
* 定义了Java编译版本为1.8
* 使用UTF-8格式编码
* 继承自spring-boor-dependencies，这里面定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号
* 执行打包操作的配置
* 自动化的资源过滤
* 自动化的插件配置
* 针对application.peoperties和application.yuml的资源过滤，包括通过profile定义的不同环境的配置文件，例如application-dev.properties和application-dev.yml

### SpringBoot打的jar包和普通的jar包有什么区别？
Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过java -jar xxx.jar命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。
> Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。
>
> 在configuration中配置classifier标识可执行的jar；下例会生成两个jar包，xxx.jar和xxx-exec.jar
>
> ```
> <plugin>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-maven-plugin</artifactId>
>     <version>2.5.5</version>
>     <configuration>
>         <classifier>exec</classifier>
>     </configuration>
> </plugin>
> ```

### 运行SpringBoot的几种方式？
* 打包用命令或者放到容器中运行
* 用Maven或Gradle插件运行
* 直接执行main方法运行

### SpringBoot的核心配置文件有哪些？
SpringBoot的核心配置文件是application和bootstrap配置文件。
application配置文件这个容易理解，主要用于Spring Boot项目的自动化配置。
bootstrap配置文件有以下几个应用场景：

* 使用Spring Cloud Config配置中心时，这时需要在bootstrap配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
* 一些固定的不能被覆盖的属性；
* 一些加密/解密的场景。

| 配置文件    |                                                              |                        |
| ----------- | ------------------------------------------------------------ | ---------------------- |
| application | 用于配置应用程序上下文<br />主要用于配置应用程序的自动化配置，比如日志、数据库、Web等 | 属性可以被其他方式覆盖 |
| bootstrap   | 用于配置引导上下文（应用程序上下文的父上下文）<br />优先级高于application<br />主要用于从外部源加载配置属性，比如Spring Cloud Config配置中心<br />还可以在本地外部配置文件中解密属性 | 属性不能被覆盖         |

